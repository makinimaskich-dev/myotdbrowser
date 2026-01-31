<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Otd Browser</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body, html { height: 100%; font-family: 'Segoe UI', sans-serif; overflow: hidden; background: #fff; }

        /* SPLASH SCREEN */
        #splash-screen {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: linear-gradient(135deg, #87CEEB, #E0F7FA);
            display: flex; flex-direction: column; align-items: center; justify-content: center;
            z-index: 10000; transition: opacity 0.5s ease;
        }

        /* HEADER */
        .header-sky {
            background: linear-gradient(to bottom, #87CEEB, #E0F7FA);
            height: 115px; display: flex; flex-direction: column;
            padding: 10px 15px; border-bottom: 2px solid #000;
        }
        .top-row { display: flex; align-items: center; justify-content: space-between; }
        .back-rect { width: 25px; height: 35px; border: 3px solid #000; background: white; cursor: pointer; }
        .up-arrow { width: 0; height: 0; border-left: 12px solid transparent; border-right: 12px solid transparent; border-bottom: 20px solid black; cursor: pointer; }
        .money-btn { font-size: 26px; cursor: pointer; color: #1b5e20; font-weight: bold; }
        .plan-badge { font-size: 10px; font-weight: bold; padding: 2px 8px; border-radius: 5px; border: 1px solid #000; }

        /* URL BAR */
        .url-bar-top { margin-top: 10px; text-align: center; }
        .url-input-top { width: 95%; padding: 6px 15px; border: 2px solid #000; border-radius: 20px; outline: none; }

        /* HOME UI */
        #home-ui { height: calc(100% - 115px); display: flex; flex-direction: column; align-items: center; justify-content: center; padding: 20px; }
        .search-box-home { width: 100%; max-width: 500px; padding: 15px 25px; border-radius: 30px; border: 1px solid #dfe1e5; margin-bottom: 20px; box-shadow: 0 1px 6px rgba(0,0,0,0.1); }
        .btn-google { background: #f8f9fa; border: 1px solid #f8f9fa; padding: 10px 20px; border-radius: 4px; cursor: pointer; font-weight: bold; }

        /* PAY MODAL */
        #google-pay-modal { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.85); display: none; z-index: 9999; align-items: center; justify-content: center; }
        .pay-box { background: white; padding: 25px; border-radius: 20px; width: 85%; max-width: 350px; text-align: center; }
        .btn-buy { width: 100%; padding: 15px; margin: 10px 0; border: none; border-radius: 10px; font-weight: bold; cursor: pointer; }

        /* BROWSER */
        #web-view { width: 100%; height: calc(100% - 115px); display: none; }
        iframe { width: 100%; height: 100%; border: none; }
    </style>
</head>
<body>

    <div id="splash-screen">
        <h1 style="font-size: 40px;">My Otd Code</h1>
        <p>Iniciando...</p>
    </div>

    <div class="header-sky">
        <div class="top-row">
            <div class="back-rect" onclick="goHome()"></div>
            <div class="up-arrow" onclick="goHome()"></div>
            <div style="text-align: center;">
                <div style="font-weight: bold;">My Otd Browser</div>
                <span id="label-plano" class="plan-badge">PLANO GRÁTIS</span>
            </div>
            <div class="icons-right" style="display: flex; gap: 15px;">
                <div class="money-btn" onclick="abrirLoja()">$</div>
                <div style="font-size: 25px;">⋮</div>
            </div>
        </div>
        <div class="url-bar-top">
            <input type="text" id="url-input-top" class="url-input-top" placeholder="https://">
        </div>
    </div>

    <div id="main-container">
        <div id="home-ui">
            <h1 style="margin-bottom: 20px;">My Otd Code</h1>
            <input type="text" id="home-search" class="search-box-home" placeholder="Pesquisar...">
            <div style="display: flex; gap: 10px;">
                <button class="btn-google" onclick="navegar('search')">Pesquisar</button>
                <button class="btn-google" onclick="navegar('luck')">Estou com sorte</button>
            </div>
        </div>
        <div id="web-view"><iframe id="frame"></iframe></div>
    </div>

    <div id="google-pay-modal">
        <div class="pay-box">
            <h3>Google Play Billing</h3>
            <button class="btn-buy" style="background: #add8e6;" onclick="processarCompra('Standard', '5.90')">STANDARD - R$ 5,90</button>
            <button class="btn-buy" style="background: #ffd700;" onclick="processarCompra('Pro', '24.99')">PRO - R$ 24,99</button>
            <p onclick="fecharLoja()" style="cursor: pointer; margin-top: 10px;">Voltar</p>
        </div>
    </div>

    <script>
        const storage = window.localStorage;
        let planoAtivo = storage.getItem('plano_browser') || 'Grátis';

        // Splash screen timer
        setTimeout(() => { document.getElementById('splash-screen').style.display = 'none'; }, 2000);

        function atualizarLayout() {
            const label = document.getElementById('label-plano');
            label.innerText = "PLANO " + planoAtivo.toUpperCase();
            label.style.background = planoAtivo === 'Pro' ? "#ffd700" : (planoAtivo === 'Standard' ? "#add8e6" : "#eee");
        }
        atualizarLayout();

        function abrirLoja() { document.getElementById('google-pay-modal').style.display = 'flex'; }
        function fecharLoja() { document.getElementById('google-pay-modal').style.display = 'none'; }

        // --- COMANDO REAL DE PAGAMENTO DA GOOGLE PLAY ---
        async function processarCompra(nomePlano, preco) {
            const idProduto = nomePlano.toLowerCase() === 'pro' ? 'id_plano_pro' : 'id_plano_standard';
            
            if (window.PaymentRequest) {
                const metodos = [{
                    supportedMethods: 'https://play.google.com/billing',
                    data: { sku: idProduto }
                }];
                const detalhes = {
                    total: { label: 'Total', amount: { currency: 'BRL', value: preco } }
                };

                try {
                    const request = new PaymentRequest(metodos, detalhes);
                    const response = await request.show();
                    // Se o Google Play aprovar:
                    await response.complete('success');
                    salvarPlano(nomePlano);
                } catch (e) {
                    // Se estiver no PC ou der erro, simulamos para você testar
                    console.log("Ambiente fora da Play Store, simulando...");
                    if(confirm("Simular pagamento de R$ " + preco + " via Google Play?")) salvarPlano(nomePlano);
                }
            } else {
                alert("Este navegador não suporta pagamentos diretos.");
            }
        }

        function salvarPlano(tipo) {
            storage.setItem('plano_browser', tipo);
            planoAtivo = tipo;
            atualizarLayout();
            fecharLoja();
            alert("Plano " + tipo + " ativado com sucesso!");
        }

        function navegar(modo) {
            let termo = document.getElementById('home-search').value || document.getElementById('url-input-top').value;
            if(!termo && modo !== 'luck') return;
            document.getElementById('home-ui').style.display = 'none';
            document.getElementById('web-view').style.display = 'block';
            let url = modo === 'luck' ? "https://www.google.com/doodles" : 
                      (termo.includes('.') ? (termo.startsWith('http') ? termo : 'https://' + termo) : "https://www.google.com/search?igu=1&q=" + encodeURIComponent(termo));
            document.getElementById('frame').src = url;
            document.getElementById('url-input-top').value = url;
        }

        function goHome() {
            document.getElementById('home-ui').style.display = 'flex';
            document.getElementById('web-view').style.display = 'none';
            document.getElementById('frame').src = '';
            document.getElementById('url-input-top').value = '';
        }
    </script>
</body>
</html>
