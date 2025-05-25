<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Simulateur de Paiement</title>
  <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>
  <style>
    body { font-family: sans-serif; text-align: center; background: #f1f1f1; padding: 20px; }
    #scanner { width: 300px; margin: auto; }
    .message { font-size: 1.2rem; margin-top: 20px; }
    .success { color: green; }
    .error { color: red; }
  </style>
</head>
<body>
  <h1>✨ Simulateur de Paiement QR ✨</h1>
  <div id="scanner"></div>
  <div class="message" id="message">Scanne un QR code de carte...</div>
  <div id="solde"></div>

  <script>
    const scanner = new Html5Qrcode("scanner");
    const qrConfig = { fps: 10, qrbox: 250 };

    const SOLDES_PAR_DEFAUT = {
      carte1: 100,
      carte2: 100,
      carte3: 100,
      carte4: 100,
      carte5: 100,
      carte6: 100,
      carte7: 100,
      carte8: 100,
      carte9: 100,
      carte10: 100,
    };

    // Initialiser soldes si pas déjà fait
    if (!localStorage.getItem("soldes")) {
      localStorage.setItem("soldes", JSON.stringify(SOLDES_PAR_DEFAUT));
    }

    function getSoldes() {
      return JSON.parse(localStorage.getItem("soldes"));
    }

    function setSoldes(soldes) {
      localStorage.setItem("soldes", JSON.stringify(soldes));
    }

    function afficherMessage(text, type = "") {
      const msgEl = document.getElementById("message");
      msgEl.textContent = text;
      msgEl.className = "message " + type;
    }

    function afficherSolde(carte, solde) {
      document.getElementById("solde").textContent = `Solde de ${carte} : ${solde} €`;
    }

    async function traiterPaiement(url) {
      try {
        const parsed = new URL(url);
        const carte = parsed.searchParams.get("user");

        if (!carte || !SOLDES_PAR_DEFAUT.hasOwnProperty(carte)) {
          afficherMessage("Carte non reconnue ❌", "error");
          return;
        }

        const soldes = getSoldes();
        const soldeActuel = soldes[carte];

        if (soldeActuel >= 10) {
          soldes[carte] -= 10;
          setSoldes(soldes);
          afficherMessage("Paiement accepté ✅", "success");
        } else {
          afficherMessage("Fonds insuffisants ❌", "error");
        }

        afficherSolde(carte, soldes[carte]);
      } catch (e) {
        afficherMessage("QR code invalide ❌", "error");
      }
    }

    scanner.start(
      { facingMode: "environment" },
      qrConfig,
      (decodedText, decodedResult) => {
        scanner.stop();
        traiterPaiement(decodedText);
        setTimeout(() => {
          scanner.start({ facingMode: "environment" }, qrConfig, (d, r) => traiterPaiement(d));
        }, 3000);
      },
      (errorMessage) => {
        console.warn(errorMessage);
      }
    );
  </script>
</body>
</html>
