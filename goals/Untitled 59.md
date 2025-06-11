const express = require('express');
const http    = require('http');
const { Server } = require('socket.io');
const jwt     = require('jsonwebtoken');

const SECRET = 'hJU7ncW+5udrBQa6MBjSxwjcZikYwVtAzUquG9jFGX8GQ4I5j0IQU2QNOEtFX3qjl4odFvAsuD6Ah7Qt57nZS>
const app    = express();
const server = http.createServer(app);
const io     = new Server(server, { cors: { origin: '*' } });

function checkWinner(board) {
  const lines = [
    [0,1,2],[3,4,5],[6,7,8],
    [0,3,6],[1,4,7],[2,5,8],
    [0,4,8],[2,4,6]
  ];
  for (const [a,b,c] of lines) {
    if (board[a] && board[a]===board[b]&&board[a]===board[c]) {
      return board[a];
    }
  }
  return board.every(cell=>cell)?'draw':null;
}

// roomId → { board, turn, players: { X: userId, O: userId }, result }
const games = new Map();

// JWT auth on socket connect
io.use((socket, next) => {
  const auth = socket.handshake.headers['authorization'] || '';
  const m = auth.match(/^Bearer (.+)$/);
  if (!m) return next(new Error('Auth error'));
  try {
    const payload = jwt.verify(m[1], SECRET);
    socket.userId = payload.user_id;
    return next();
  } catch {
    return next(new Error('Auth error'));
  }
});

io.on('connection', socket => {
  // Join room
  socket.on('join_room', ({ roomId, mark }) => {
    if (!games.has(roomId)) {
      games.set(roomId,{
        board:  Array(9).fill(null),
        turn:   'X',
        players:{ X:null, O:null },
        result: null
      });
    }
    const game = games.get(roomId);
    // Assign player slot
    game.players[mark] = socket.userId;
    socket.join(roomId);
    socket.emit('room_joined', { roomId });
    io.in(roomId).emit('game_update', {
      board:  game.board,
      turn:   game.turn,
      result: game.result
    });
  });

  // Make move
  socket.on('make_move', ({ roomId, index, mark }) => {
    const game = games.get(roomId);
    if (!game || game.result) return;
    if (game.turn!==mark || game.board[index]!==null) return;
    // Only the correct user can move
    if (game.players[mark] !== socket.userId) return;

    game.board[index] = mark;
    game.turn = mark==='X'?'O':'X';
    game.result = checkWinner(game.board);

    io.in(roomId).emit('game_update', {
      board:  game.board,
      turn:   game.turn,
      result: game.result
    });
    if (game.result) {
      io.in(roomId).emit('game_over', { result: game.result });
    }
  });

  // Reset
  socket.on('reset_game', ({ roomId }) => {
    const game = games.get(roomId);
    if (!game) return;
     game.board = Array(9).fill(null);
    game.turn  = 'X';
    game.result= null;
    io.in(roomId).emit('game_update', {
      board:  game.board,
      turn:   game.turn,
      result: game.result
    });
  });
});

server.listen(3000, () => {
  console.log('Tic-Tac-Toe socket server listening on port 3000');
});

