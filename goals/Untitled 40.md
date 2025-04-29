<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Mediathek Example</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">

  <style>
    /* Basic page layout */
    body {
      font-family: sans-serif;
      margin: 20px;
      background-color: #f9f9f9;
    }
    h1 {
      margin-bottom: 20px;
    }

    /* Category Section */
    .mediathek-category {
      margin-bottom: 40px;
    }
    .mediathek-category h2 {
      margin-bottom: 16px;
      font-size: 1.4rem;
      font-weight: 600;
    }

    /* Responsive Video Grid */
    .video-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
      gap: 16px;
    }

    /* Video Wrapper */
    .video-wrapper {
      cursor: pointer;
      text-align: left; /* Ensure child elements align left */
    }

    /* Video Thumbnail Card (16:9) */
    .video-card {
      width: 100%;
      aspect-ratio: 16 / 9; /* Ensures correct aspect ratio if supported */
      background-color: #ccc;
      display: flex;
      align-items: center;
      justify-content: center;
      border-radius: 4px;
      overflow: hidden; /* ensures the image is clipped within border radius */
    }
    /* Make sure thumbnail images fill the .video-card correctly */
    .video-card img {
      width: 100%;
      height: 100%;
      object-fit: cover;
      display: block;
    }

    /* Video Title - left-aligned */
    .video-title {
      margin-top: 8px;
      font-size: 0.9rem;
      color: #333;
      text-align: left;
    }
    .video-title::before {
      content: "• ";
      color: #666;
    }

    /* Modal overlay: full window, black background */
    .modal-overlay {
      position: fixed;
      inset: 0; /* top:0; right:0; bottom:0; left:0; */
      background: #000; /* fully black */
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 1000;
    }

    /* Modal content: occupy the full overlay */
    .modal-content {
      position: relative;
      width: 100%;
      height: 100%;
      display: flex;
      align-items: center;
      justify-content: center;
      /* No extra padding/border so video can go edge-to-edge */
    }

    /* Full-window player container */
    .video-player-container {
      position: relative;
      width: 100%;
      height: 100%;
      background-color: #222; /* fallback behind video */
      overflow: hidden;
    }
    .video-player-container video {
      width: 100%;
      height: 100%;
      object-fit: cover; /* or 'contain' if you want the entire frame visible */
      display: block;
    }

    /* Close button (X) overlaid on top of the video, top-right corner */
    .modal-close {
      position: absolute;
      top: 20px;
      right: 20px;
      cursor: pointer;
      font-size: 2rem;
      color: #fff;
      z-index: 1101; /* above the video */
    }

    /* Bottom green bar for custom controls */
    .controls-bar {
      position: absolute;
      bottom: 0;
      left: 0;
      right: 0;
      background-color: #9cd137; /* light green */
      display: flex;
      align-items: center;
      padding: 8px;
      box-sizing: border-box;
    }

    /* Buttons (play/pause, fullscreen) */
    .control-btn {
      background: none;
      border: none;
      font-size: 1.2rem;
      color: #333;
      cursor: pointer;
      margin: 0 8px;
      user-select: none;
    }
    .control-btn:hover {
      color: #111;
    }

    /* Progress container + slider */
    .progress-container {
      flex: 1;
      margin: 0 8px;
      position: relative;
    }
    .progress-bar {
      width: 100%;
      cursor: pointer;
      background: transparent;
      -webkit-appearance: none; /* Safari */
      appearance: none;
    }
    .progress-bar::-webkit-slider-runnable-track {
      height: 4px;
      background: #666;
      border-radius: 2px;
    }
    .progress-bar::-webkit-slider-thumb {
      -webkit-appearance: none;
      height: 12px;
      width: 12px;
      border-radius: 50%;
      background: #333;
      margin-top: -4px;
      cursor: pointer;
    }
    .progress-bar::-moz-range-track {
      height: 4px;
      background: #666;
      border-radius: 2px;
    }
    .progress-bar::-moz-range-thumb {
      height: 12px;
      width: 12px;
      border-radius: 50%;
      background: #333;
      cursor: pointer;
    }

    /* Mobile adjustments if needed */
    @media (max-width: 768px) {
      body {
        margin: 0; /* optionally remove margins on small screens */
      }
    }
  </style>
</head>
<body>

  <h1>Mediathek</h1>
  <div id="mediathek"></div>

  <script>
    /* Example data with thumbnails */
    const data = {
      "categories": [
        {
          "category": "Technik",
          "linklist": [
            {
              "title": "Video 1",
              "url": "http://fahrschule-peppermint.com/wp-content/uploads/2025/03/Online-Service-PP-Start-in-die-Fahrprufung_1.mp4",
              "thumbnail": "http://fahrschule-peppermint.com/wp-content/uploads/2025/03/Online-Service-PP-Start-in-die-Fahrprufung.png"
            },
            {
              "title": "Video 2",
              "url": "http://fahrschule-peppermint.com/wp-content/uploads/2025/03/Online-Service-PP-Prufgebiet-Mchn-Ramersdorf-1_1.mp4",
              "thumbnail": "http://fahrschule-peppermint.com/wp-content/uploads/2025/03/Online-Service-PP-Prufgebiet-Mchn-Ramersdorf-1.png"
            },
            {
              "title": "Video 3",
              "url": "http://fahrschule-peppermint.com/wp-content/uploads/2025/03/Online-Service-PP-Prufgebiet-Autobahn-richtig-befahren_1.mp4",
              "thumbnail": "https://via.placeholder.com/320x180?text=Video+3"
            },
            {
              "title": "Video 4",
              "url": "http://fahrschule-peppermint.com/wp-content/uploads/2025/03/Online-Service-PP-EPP_1.mp4",
              "thumbnail": "https://via.placeholder.com/320x180?text=Video+4"
            }
          ]
        },
        {
          "category": "Grundfahraufgaben",
          "linklist": [
            { 
              "title": "Video 5", 
              "url": "#",
              "thumbnail": "https://via.placeholder.com/320x180?text=Video+5"
            },
            { 
              "title": "Video 6", 
              "url": "#",
              "thumbnail": "https://via.placeholder.com/320x180?text=Video+6"
            },
            { 
              "title": "Video 7", 
              "url": "#",
              "thumbnail": "https://via.placeholder.com/320x180?text=Video+7"
            },
            { 
              "title": "Video 8", 
              "url": "#",
              "thumbnail": "https://via.placeholder.com/320x180?text=Video+8"
            }
          ]
        }
      ]
    };

    const mediathekContainer = document.getElementById("mediathek");

    /**
     * Show a fullscreen modal with the custom controls
     */
    function openModal(videoUrl) {
      // Overlay to cover entire screen
      const overlay = document.createElement("div");
      overlay.className = "modal-overlay";

      // Modal content: full width/height
      const modalContent = document.createElement("div");
      modalContent.className = "modal-content";

      // "X" close button overlaid on video
      const closeButton = document.createElement("div");
      closeButton.className = "modal-close";
      closeButton.textContent = "X";
      closeButton.addEventListener("click", closeModal);

      // Video player container
      const playerContainer = document.createElement("div");
      playerContainer.className = "video-player-container";

      // Actual video element (hide native controls)
      const videoElem = document.createElement("video");
      videoElem.src = videoUrl;
      videoElem.controls = false; 
      videoElem.setAttribute("playsinline", ""); // better for mobile

      // Bottom green bar for custom controls
      const controlsBar = document.createElement("div");
      controlsBar.className = "controls-bar";

      // Play/Pause button
      const playPauseBtn = document.createElement("button");
      playPauseBtn.className = "control-btn";
      playPauseBtn.textContent = "►";

      // Progress slider container
      const progressContainer = document.createElement("div");
      progressContainer.className = "progress-container";
      const progressBar = document.createElement("input");
      progressBar.type = "range";
      progressBar.min = 0;
      progressBar.value = 0;
      progressBar.className = "progress-bar";
      progressContainer.appendChild(progressBar);

      // Fullscreen button
      const fullscreenBtn = document.createElement("button");
      fullscreenBtn.className = "control-btn";
      fullscreenBtn.textContent = "⤢";

      // Append controls
      controlsBar.appendChild(playPauseBtn);
      controlsBar.appendChild(progressContainer);
      controlsBar.appendChild(fullscreenBtn);

      // Put video + control bar inside container
      playerContainer.appendChild(videoElem);
      playerContainer.appendChild(controlsBar);

      // Add everything into modal
      modalContent.appendChild(playerContainer);
      modalContent.appendChild(closeButton);

      // Put modal into overlay, then into document
      overlay.appendChild(modalContent);
      document.body.appendChild(overlay);

      // Clicking outside the video closes modal
      overlay.addEventListener("click", function (e) {
        if (e.target === overlay) {
          closeModal();
        }
      });

      // Close function
      function closeModal() {
        document.body.removeChild(overlay);
      }

      /* Custom Control Logic */

      // Play/Pause
      playPauseBtn.addEventListener("click", () => {
        if (videoElem.paused) {
          videoElem.play();
          playPauseBtn.textContent = "❚❚"; // pause symbol
        } else {
          videoElem.pause();
          playPauseBtn.textContent = "►"; // play symbol
        }
      });

      // Update progress bar in real-time
      videoElem.addEventListener("timeupdate", () => {
        if (!videoElem.duration) return;
        progressBar.value = (videoElem.currentTime / videoElem.duration) * 100;
      });

      // Seeking
      progressBar.addEventListener("input", () => {
        const seekTime = (progressBar.value / 100) * videoElem.duration;
        videoElem.currentTime = seekTime;
      });

      // Fullscreen toggle
      fullscreenBtn.addEventListener("click", () => {
        if (!document.fullscreenElement) {
          playerContainer.requestFullscreen && playerContainer.requestFullscreen();
        } else {
          document.exitFullscreen && document.exitFullscreen();
        }
      });
    }

    /**
     * Build the category/video grid
     */
    data.categories.forEach(categoryObj => {
      const categoryWrapper = document.createElement("div");
      categoryWrapper.className = "mediathek-category";

      const categoryTitle = document.createElement("h2");
      categoryTitle.textContent = categoryObj.category;
      categoryWrapper.appendChild(categoryTitle);

      const videoGrid = document.createElement("div");
      videoGrid.className = "video-grid";

      categoryObj.linklist.forEach(video => {
        const videoWrapper = document.createElement("div");
        videoWrapper.className = "video-wrapper";

        const videoCard = document.createElement("div");
        videoCard.className = "video-card";

        // If a valid thumbnail URL, show it. Otherwise, show fallback text.
        if (video.thumbnail) {
          const thumbnailImg = document.createElement("img");
          thumbnailImg.src = video.thumbnail;
          thumbnailImg.alt = video.title;
          videoCard.appendChild(thumbnailImg);
        } else {
          videoCard.textContent = "No thumbnail";
        }

        // Click to open modal if valid URL
        if (video.url && video.url !== "#") {
          videoWrapper.addEventListener("click", () => {
            openModal(video.url);
          });
        }

        const videoTitle = document.createElement("div");
        videoTitle.className = "video-title";
        videoTitle.textContent = video.title;

        videoWrapper.appendChild(videoCard);
        videoWrapper.appendChild(videoTitle);
        videoGrid.appendChild(videoWrapper);
      });

      categoryWrapper.appendChild(videoGrid);
      mediathekContainer.appendChild(categoryWrapper);
    });
  </script>
</body>
</html>
