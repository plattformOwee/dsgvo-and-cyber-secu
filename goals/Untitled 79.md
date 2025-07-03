<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Mediathek Example</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">

  <style>
    :root {
      --card-h: 180px;
      --card-w: calc(16 / 9 * var(--card-h));
      --gap: 50px;
    }

    body {
      font-family: Arial, sans-serif;
      margin: 20px;
      background-color: #f9f9f9;
      color: #444;
    }

    h1 {
      margin-bottom: 20px;
      font-weight: 700;
      color: #444;
    }

    .mediathek-category {
      margin-bottom: 40px;
    }

    .mediathek-category h2 {
      margin-bottom: 16px;
      font-size: 1.4rem;
      font-weight: 700;
      font-family: Arial, sans-serif;
      color: #444;
    }

    .video-grid {
      display: flex;
      flex-wrap: wrap;
      gap: var(--gap);
    }

    .video-wrapper {
      width: var(--card-w);
      cursor: pointer;
      text-align: left;
    }

    .video-card {
      width: 100%;
      height: var(--card-h);
      background-color: #ccc;
      display: flex;
      align-items: center;
      justify-content: center;
      border-radius: 4px;
      overflow: hidden;
    }

    .video-card img {
      width: 100%;
      height: 100%;
      object-fit: cover;
      display: block;
      border-radius: 4px;
    }

    .video-title {
      margin-top: 8px;
      font-size: 1rem;
      color: #444;
      text-align: left;
      font-family: Arial, sans-serif;
      font-weight: 700;
    }

    /* MOBILE OVERRIDES */
    @media (max-width: 450px) {
      /* Let the card height follow the width, keep 16:9 aspect ratio */
      .video-card {
        height: auto !important;
        aspect-ratio: 16 / 9;
      }

      /* Full-width wrappers with 5px side padding */
      .video-wrapper {
        width: 100% !important;
        padding: 0 5px;
        box-sizing: border-box;
      }
    }

    .modal-overlay {
      position: fixed;
      inset: 0;
      background: #000;
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 1000;
    }

    .modal-content {
      position: relative;
      width: 100%;
      height: 100%;
      display: flex;
      align-items: center;
      justify-content: center;
    }

    .video-player-container {
      position: relative;
      width: 100%;
      height: 100%;
      background-color: #222;
      overflow: hidden;
    }

    .video-player-container video {
      width: 100%;
      height: 100%;
      object-fit: cover;
      display: block;
    }

    .modal-close {
      position: absolute;
      top: 20px;
      right: 20px;
      cursor: pointer;
      font-size: 2rem;
      color: #fff;
      z-index: 1101;
    }

    .controls-bar {
      position: absolute;
      bottom: 0;
      left: 0;
      right: 0;
      background-color: #9cd137;
      display: flex;
      align-items: center;
      padding: 8px;
      box-sizing: border-box;
    }

    .control-btn {
      background: none;
      border: none;
      font-size: 1.2rem;
      color: #333;
      cursor: pointer;
      margin: 0 8px;
      user-select: none;
    }

    .control-btn:hover { color: #111; }

    .progress-container {
      flex: 1;
      margin: 0 8px;
      position: relative;
    }

    .progress-bar {
      width: 100%;
      cursor: pointer;
      background: transparent;
      -webkit-appearance: none;
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
  </style>
</head>
<body>

  <h1>Mediathek</h1>
  <div id="mediathek"></div>

  <script>
    const data = {
      categories: [
        {
          category: "PP-Inside",
          linklist: [
            {
              title: "Der wahre Wert deines Führerscheins",
              url: "http://fahrschule-peppermint.com/wp-content/uploads/2025/06/Der-wahre-Wert-deines-Fuhrerscheins.mp4",
              thumbnail: "http://fahrschule-peppermint.com/wp-content/uploads/2025/06/Online-Service-PP-Der-wahre-Wert-deines-Fuhrerscheins.png"
            },
            {
              title: "Wahl der richtigen Fahrschule",
              url: "http://fahrschule-peppermint.com/wp-content/uploads/2025/06/Wahl-der-richtigen-Fahrschule.mp4",
              thumbnail: "http://fahrschule-peppermint.com/wp-content/uploads/2025/06/WhatsApp-Bild-2025-06-11-um-19.54.51_037d08ce.jpg"
            },
            {
              title: "Intensivkurs",
              url: "http://fahrschule-peppermint.com/wp-content/uploads/2025/06/WhatsApp-Video-2025-06-10-um-17.15.24_2457b80f.mp4",
              thumbnail: "http://fahrschule-peppermint.com/wp-content/uploads/2025/06/Online-Service-PP-Intensivkurs.png"
            }
          ]
        },
        {
          category: "PP-Praxisprüfung",
          linklist: [
            {
              title: "Autobahn richtig befahren",
              url: "http://fahrschule-peppermint.com/wp-content/uploads/2025/03/Online-Service-PP-Prufgebiet-Autobahn-richtig-befahren_1.mp4",
              thumbnail: "http://fahrschule-peppermint.com/wp-content/uploads/2025/03/Online-Service-PP-Prufgebiet-Autobahn-richtig-befahren.png"
            },
            {
              title: "Prüfgebiet München Ramersdorf",
              url: "http://fahrschule-peppermint.com/wp-content/uploads/2025/03/Online-Service-PP-Prufgebiet-Mchn-Ramersdorf-1_1.mp4",
              thumbnail: "http://fahrschule-peppermint.com/wp-content/uploads/2025/03/Online-Service-PP-Prufgebiet-Mchn-Ramersdorf-1.png"
            },
            {
              title: "Prüfgebiet Taufkirchen",
              url: "http://fahrschule-peppermint.com/wp-content/uploads/2025/06/Online-Service-PP-Prufgebiet-Autobahn-A995-Taufkirchen-West.png",
              thumbnail: "http://fahrschule-peppermint.com/wp-content/uploads/2025/06/Online-Service-PP-Prufgebiet-Autobahn-A995-Taufkirchen-West.png"
            },
            {
              title: "Start in die Fahrprüfung",
              url: "http://fahrschule-peppermint.com/wp-content/uploads/2025/03/Online-Service-PP-Start-in-die-Fahrprufung_1.mp4",
              thumbnail: "http://fahrschule-peppermint.com/wp-content/uploads/2025/03/Online-Service-PP-Start-in-die-Fahrprufung.png"
            },
            {
              title: "Prüfprotokoll",
              url: "http://fahrschule-peppermint.com/wp-content/uploads/2025/03/Online-Service-PP-EPP_1.mp4",
              thumbnail: "http://fahrschule-peppermint.com/wp-content/uploads/2025/03/Online-Service-PP-Fahrprufung-EPP.png"
            }
          ]
        },
        {
          category: "PP-Theorieprüfung",
          linklist: [
            {
              title: "Theorieprüfung: Bildfragen",
              url: "http://fahrschule-peppermint.com/wp-content/uploads/2025/03/Online-Service-PP-Theorieprufung-Bildfragen.mp4",
              thumbnail: "http://fahrschule-peppermint.com/wp-content/uploads/2025/03/Online-Service-PP-Theorieprufung-Bildfragen.png"
            },
            {
              title: "Theorieprüfung: Videofragen",
              url: "http://fahrschule-peppermint.com/wp-content/uploads/2025/03/Online-Service-PP-Theorieprufung-Videofragen.mp4",
              thumbnail: "http://fahrschule-peppermint.com/wp-content/uploads/2025/03/Online-Service-PP-Theorieprufung-Videofragen.png"
            }
          ]
        },
        {
          category: "PP-Technik",
          linklist: [
            {
              title: "Technik: Der Autoreifen",
              url: "http://fahrschule-peppermint.com/wp-content/uploads/2025/03/Online-Service-PP-Der-Autoreifen_1.mp4",
              thumbnail: "http://fahrschule-peppermint.com/wp-content/uploads/2025/03/Online-Service-PP-Der-Autoreifen.png"
            },
            {
              title: "Technik: Das Kühlsystem",
              url: "http://fahrschule-peppermint.com/wp-content/uploads/2025/05/Online-Service-PP-Das-Kuhlsystem.mp4",
              thumbnail: "http://fahrschule-peppermint.com/wp-content/uploads/2025/05/Online-Service-PP-Kuhlsystem.png"
            },
            {
              title: "Technik: Die Bremse",
              url: "http://fahrschule-peppermint.com/wp-content/uploads/2025/05/Onlinr-Service-Die-Bremse.mp4",
              thumbnail: "http://fahrschule-peppermint.com/wp-content/uploads/2025/05/Online-Service-PP-Die-Bremse.png"
            }
          ]
        },
        {
          category: "PP-Straßenverkehrssystem",
          linklist: [
            {
              title: "Wiki Verkehrsschilder",
              url: "http://fahrschule-peppermint.com/wp-content/uploads/2025/06/Online-Service-PP-Unterwegs-im-Schilderwald.mp4",
              thumbnail: "http://fahrschule-peppermint.com/wp-content/uploads/2025/06/Online-Service-PP-Verkehrszeichen-Im-Schilderwald.png"
            }
          ]
        }
      ]
    };

    const mediathekContainer = document.getElementById("mediathek");

    function openModal(videoUrl) {
      const overlay = document.createElement("div");
      overlay.className = "modal-overlay";

      const modalContent = document.createElement("div");
      modalContent.className = "modal-content";

      const closeButton = document.createElement("div");
      closeButton.className = "modal-close";
      closeButton.textContent = "X";
      closeButton.addEventListener("click", closeModal);

      const playerContainer = document.createElement("div");
      playerContainer.className = "video-player-container";

      const videoElem = document.createElement("video");
      videoElem.src = videoUrl;
      videoElem.controls = false;
      videoElem.setAttribute("playsinline", "");

      const controlsBar = document.createElement("div");
      controlsBar.className = "controls-bar";

      const playPauseBtn = document.createElement("button");
      playPauseBtn.className = "control-btn";
      playPauseBtn.textContent = "►";

      const progressContainer = document.createElement("div");
      progressContainer.className = "progress-container";
      const progressBar = document.createElement("input");
      progressBar.type = "range";
      progressBar.min = 0;
      progressBar.value = 0;
      progressBar.className = "progress-bar";
      progressContainer.appendChild(progressBar);

      const fullscreenBtn = document.createElement("button");
      fullscreenBtn.className = "control-btn";
      fullscreenBtn.textContent = "⤢";

      controlsBar.appendChild(playPauseBtn);
      controlsBar.appendChild(progressContainer);
      controlsBar.appendChild(fullscreenBtn);

      playerContainer.appendChild(videoElem);
      playerContainer.appendChild(controlsBar);

      modalContent.appendChild(playerContainer);
      modalContent.appendChild(closeButton);
      overlay.appendChild(modalContent);
      document.body.appendChild(overlay);

      overlay.addEventListener("click", function (e) {
        if (e.target === overlay) {
          closeModal();
        }
      });

      function closeModal() {
        document.body.removeChild(overlay);
      }

      playPauseBtn.addEventListener("click", () => {
        if (videoElem.paused) {
          videoElem.play();
          playPauseBtn.textContent = "❚❚";
        } else {
          videoElem.pause();
          playPauseBtn.textContent = "►";
        }
      });

      videoElem.addEventListener("timeupdate", () => {
        if (!videoElem.duration) return;
        progressBar.value = (videoElem.currentTime / videoElem.duration) * 100;
      });

      progressBar.addEventListener("input", () => {
        const seekTime = (progressBar.value / 100) * videoElem.duration;
        videoElem.currentTime = seekTime;
      });

      fullscreenBtn.addEventListener("click", () => {
        if (!document.fullscreenElement) {
          playerContainer.requestFullscreen && playerContainer.requestFullscreen();
        } else {
          document.exitFullscreen && document.exitFullscreen();
        }
      });
    }

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

        if (video.thumbnail) {
          const thumbnailImg = document.createElement("img");
          thumbnailImg.src = video.thumbnail;
          thumbnailImg.alt = video.title;
          videoCard.appendChild(thumbnailImg);
        } else {
          videoCard.textContent = "No thumbnail";
        }

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
