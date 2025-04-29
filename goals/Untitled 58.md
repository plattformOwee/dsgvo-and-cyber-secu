<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Continuous Scrolling Text Animation</title>
    <style>
        .marquee-container {
            overflow: hidden;
            white-space: nowrap;
            box-sizing: border-box;
            width: 100%;
            background-color: #B9DF90;
            color: #ffffff;
            padding: 15px;
            font-family: Arial, sans-serif;
            display: flex;
        }

        .marquee-text {
            display: inline-block;
            animation: marquee 300s linear infinite;
        }

        .marquee-content {
            display: inline-block;
            padding-right: 50px;
        }

        @keyframes marquee {
            0% { transform: translateX(0); }
            100% { transform: translateX(-100%); }
        }

        /* Responsive font sizes */
        @media (max-width: 600px) {
            .marquee-container {
                font-size: 13px;
				padding: 4px;/* Small screens */
            }
        }

        @media (min-width: 601px) and (max-width: 1024px) {
            .marquee-container {
                font-size: 20px; /* Medium screens */
            }
        }

        @media (min-width: 1025px) {
            .marquee-container {
                font-size: 34px; /* Large screens */
            }
        }
    </style>
</head>
<body>
    <div class="marquee-container">
        <div class="marquee-text">
            <div class="marquee-content">
____________________________________________________NEWS !  📰 Der Führerschein wird billiger, Führerscheinreform im Koalitionsvertrag. Wir sind bereit. Während bundesweit noch über die neue Führerscheinreform diskutiert wird, ist die Fahrschule Peppermint schon einen Schritt weiter. Moderne Ausbildungsmethoden, ein moderner Fahrsimulator, digitale Tools, flexible Lernkonzepte – was bald Standard wird, ist bei uns längst Realität. Wieso warten? Jetzt einsteigen und schon heute von der Zukunft profitieren! Auto und Motorrad: B, B197, B78, BE, B196, A1, A2, A, AM, Mofa. Anmeldung jederzeit online oder in der Fahrschule in der Falkenstr. 5, 81541 München. Ob Motorrad, Auto oder Anhänger, bei uns finden Sie die passenden Kurse für Ihre Bedürfnisse. Unser erfahrenes Team sorgt dafür, dass Sie sicher und effizient Ihre Fahrerlaubnis erwerben.
            </div>
            <div class="marquee-content">
____________________________________________________NEWS! 📰 Der Führerschein wird billiger, Führerscheinreform im Koalitionsvertrag. Wir sind bereit. Während bundesweit noch über die neue Führerscheinreform diskutiert wird, ist die Fahrschule Peppermint schon einen Schritt weiter. Moderne Ausbildungsmethoden, ein moderner Fahrsimulator, digitale Tools, flexible Lernkonzepte – was bald Standard wird, ist bei uns längst Realität. Wieso warten? Jetzt einsteigen und schon heute von der Zukunft profitieren! Auto und Motorrad: B, B197, B78, BE, B196, A1, A2, A, AM, Mofa. Anmeldung jederzeit online oder in der Fahrschule in der Falkenstr. 5, 81541 München. Ob Motorrad, Auto oder Anhänger, bei uns finden Sie die passenden Kurse für Ihre Bedürfnisse. Unser erfahrenes Team sorgt dafür, dass Sie sicher und effizient Ihre Fahrerlaubnis erwerben.
            </div>
        </div>
    </div>
</body>
</html>
