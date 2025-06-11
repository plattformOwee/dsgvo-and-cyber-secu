<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Fahrschule Peppermint – Intensivkurs</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 20px;
            line-height: 1.6;
            background-color: #fff;
            color: #333;
        }
        h1 {
            font-size: 2rem;
            margin-bottom: 0.5rem;
        }
        p {
            margin-bottom: 1rem;
        }
        .cta-buttons {
            margin: 1.5rem 0;
            display: flex;
            gap: 1rem;
            flex-wrap: wrap;
        }
        .button {
            display: inline-block;
            padding: 0.75rem 1.5rem;
            font-size: 1rem;
            text-decoration: none;
            background-color: #00B300;
            color: #fff;
            border-radius: 0.5rem;
            transition: background-color 0.3s ease;
        }
        .button:hover {
            background-color: #006600;
        }
        /* Menü-Styles aus Ihrem Template: */
        .menu-container {
            display: flex;
            justify-content: flex-end;
            align-items: center;
            margin-bottom: 2rem;
        }
        .menu-wrapper { display: flex; align-items: center; }
        .toggle-menu { position: relative; z-index: 10; }
        .toggle-menu input { display: none; }
        .menu-icon-label {
            display: inline-block;
            width: 50px;
            height: 50px;
            cursor: pointer;
            background: url('http://fahrschule-peppermint.com/wp-content/uploads/2024/09/Komponente-55-%E2%80%93-1.svg') no-repeat center center;
            background-size: contain;
        }
        .side-menu {
            position: fixed;
            top: 0; right: 0;
            height: 100%; width: 250px;
            background: linear-gradient(145deg, #fff, #f5f5f5);
            box-shadow: -2px 0 10px rgba(0,0,0,0.1);
            padding: 20px 0;
            transform: translateX(100%);
            transition: transform 0.4s ease;
            z-index: 20;
            display: flex; flex-direction: column;
        }
        .side-menu a {
            display: flex;
            align-items: center;
            padding: 12px 20px;
            font-size: 1rem;
            color: #333;
            text-decoration: none;
            border-left: 4px solid transparent;
            transition: all 0.3s ease;
        }
        .side-menu a:hover {
            background-color: #98FF98;
            border-left: 4px solid #00B300;
            color: #006600;
        }
        .side-menu a i { margin-right: 10px; color: #00B300; }
        .close-button {
            position: absolute; top: 15px; left: 15px;
            background: none; border: none;
            font-size: 1.5rem; cursor: pointer;
        }
        .toggle-menu input:checked ~ .side-menu {
            transform: translateX(0);
        }
        .calendly-popup-overlay {
            position: fixed; top: 0; left: 0;
            width: 100%; height: 100%;
            background-color: rgba(0,0,0,0.6);
            display: none; justify-content: center; align-items: center;
            z-index: 30;
        }
        .calendly-popup {
            background: #fff;
            width: 90%; max-width: 600px;
            height: 700px;
            border-radius: 10px;
            overflow: hidden;
            position: relative;
        }
        .calendly-close-btn {
            position: absolute; top: 10px; right: 10px;
            font-size: 2rem; background: none; border: none;
            color: #333; cursor: pointer;
            z-index: 40;
        }
        @media (max-width: 480px) {
            .menu-icon-label { width: 35px; height: 35px; }
            .side-menu a { font-size: 0.9rem; }
            .calendly-close-btn { font-size: 1.5rem; top: 5px; right: 5px; }
        }
    </style>
</head>
<body>

    <!-- Hamburger-Menü -->
    <div class="menu-container">
        <div class="menu-wrapper">
            <div class="toggle-menu">
                <input type="checkbox" id="menu-trigger">
                <label for="menu-trigger" class="menu-icon-label"></label>
                <nav class="side-menu">
                    <button class="close-button" onclick="document.getElementById('menu-trigger').checked = false;">&#10005;</button>
                    <a href="https://fahrschule-peppermint.com/"><i>🏠</i>Home</a>
                    <a href="#" id="menu-anmeldung"><i>📝</i>Anmeldung</a>
                    <a href="#" id="menu-probefahrt"><i>🚘</i>Probefahrt</a>
                    <a href="#" id="menu-termin"><i>📅</i>Termin</a>
                    <a href="https://fahrschule-peppermint.com/mediathek/"><i>📺</i>Mediathek</a>
                    <a href="https://fahrschule-peppermint.com/infos-fur-eltern-2/"><i>👨‍👩‍👧‍👦</i>Infos für Eltern</a>
                    <a href="https://fahrschule-peppermint.com/fuhrerscheine-peppermint-2/"><i>🪪</i>Führerscheine</a>
                    <a href="https://fahrschule-peppermint.com/kostenkalkulator/"><i>💶</i>Kostenkalkulator</a>
                    <a href="https://fahrschule-peppermint.com/blog/"><i>🗒️</i>Blog</a>
                    <a href="https://fahrschule-peppermint.com/galerie-2/"><i>🖼️</i>Impressionen</a>
                    <a href="https://fahrschule-peppermint.com/kontaktformular/"><i>📞</i>Kontakt</a>
                    <a href="https://fahrschule-peppermint.com/agb/"><i>📄</i>AGB</a>
                    <a href="https://fahrschule-peppermint.com/impressum/"><i>ℹ️</i>Impressum</a>
                </nav>
            </div>
        </div>
    </div>

    <!-- Haupt-Text -->
    <h1>Intensivkurs Führerschein</h1>
    <p>Du möchtest deinen Führerschein schnell und effizient machen? Dann ist unser Intensivkurs genau das Richtige für dich! In nur wenigen Wochen zum Ziel – <em>drive your life!</em></p>
    <p>Voraussetzung für eine erfolgreiche Teilnahme:</p>
    <ul>
        <li>Anmeldung bei der Fahrschule Peppermint</li>
        <li>Erste-Hilfe-Kurs</li>
        <li>Sehtest</li>
    </ul>
    <p>Im Vorfeld absolvierst du ganz flexibel deine Theoriestunden und trainierst stressfrei am Fahrsimulator. So werden deine Fahrstunden richtig breezy!</p>
    <p>Starte jetzt durch mit dem Intensivkurs der Fahrschule Peppermint.</p>

    <!-- Call-to-Action Buttons -->
    <div class="cta-buttons">
        <a href="https://fahrschule-peppermint.com/intensivkurs-auto-kosten/" class="button">Preise</a>
        <a href="#" id="calendly-trigger-main" class="button">Termin</a>
        <a href="https://fahrschule-peppermint.com/form2/" class="button">Kontakt</a>
    </div>

    <!-- Calendly Popup Overlay -->
    <div class="calendly-popup-overlay" id="calendly-popup-overlay">
        <div class="calendly-popup">
            <button class="calendly-close-btn" onclick="closeCalendlyPopup()">&#10005;</button>
            <div class="calendly-inline-widget"
                 data-url="https://calendly.com/fahrschule-peppermint/probefahrt-simulator"
                 style="min-width:320px;height:100%;"></div>
        </div>
    </div>

    <script src="https://assets.calendly.com/assets/external/widget.js" async></script>
    <script>
        // Kalender-Popup öffnen
        const calendlyOverlay = document.getElementById('calendly-popup-overlay');
        function openCalendlyPopup() {
            calendlyOverlay.style.display = 'flex';
        }
        function closeCalendlyPopup() {
            calendlyOverlay.style.display = 'none';
        }

        // Buttons & Menü-Links verbinden
        document.getElementById('calendly-trigger-main')
                .addEventListener('click', function(e){ e.preventDefault(); openCalendlyPopup(); });

        document.getElementById('menu-termin')
                .addEventListener('click', function(e){ e.preventDefault(); openCalendlyPopup(); });

        document.getElementById('menu-probefahrt')
                .addEventListener('click', function(e){ e.preventDefault(); openCalendlyPopup(); });

        document.getElementById('menu-anmeldung')
                .addEventListener('click', function(e){
                    // z.B. zum Anmeldeformular scrollen oder öffnen
                    window.location.href = "#"; 
                });
    </script>
</body>
</html>
