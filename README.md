<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GuguBank</title>
    <!-- Incluindo Tailwind CSS para estilização moderna e responsiva -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Incluindo a biblioteca jsPDF para geração de PDF -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://unpkg.com/jspdf-autotable@3.8.1/dist/jspdf.plugin.autotable.js"></script>
    <style>
        /* Definindo uma fonte moderna para o corpo do documento */
        body {
            font-family: 'Inter', sans-serif;
        }
        /* Estilos adicionais para a lista de transações */
        .transaction-list {
            list-style-type: none;
            padding: 0;
            margin: 0;
        }
        .transaction-item.loan {
            border-left: 4px solid #ef4444; /* Vermelho para empréstimos */
        }
        .transaction-item.payment {
            border-left: 4px solid #22c55e; /* Verde para pagamentos */
        }
        /* Estilos para os botões de seleção de categoria */
        .category-button {
            transition: transform 0.2s, box-shadow 0.2s;
            cursor: pointer;
        }
        .category-button:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }
    </style>
</head>
<body class="bg-gray-100 flex items-center justify-center min-h-screen p-4">

    <!-- Container principal do aplicativo -->
    <div class="bg-white p-6 md:p-8 rounded-2xl shadow-xl w-full max-w-2xl transition-transform transform">
        <h1 class="text-3xl md:text-4xl font-extrabold text-center text-gray-800 mb-6 tracking-wide">
            GuguBank
        </h1>

        <!-- Seção da Visão Geral (Tela 1) -->
        <div id="summary-view" class="space-y-6">
            <div class="bg-gray-50 p-4 rounded-lg border border-gray-200">
                <h2 class="text-xl font-bold text-gray-700 mb-4">Saldo Total</h2>
                <div class="flex justify-between items-center mb-4">
                    <p class="text-lg font-bold text-gray-600">A Receber:</p>
                    <span id="to-receive-balance" class="font-extrabold text-green-600 text-xl">R$ 0,00</span>
                </div>
                <div class="flex justify-between items-center">
                    <p class="text-lg font-bold text-gray-600">A Pagar:</p>
                    <span id="to-pay-balance" class="font-extrabold text-red-600 text-xl">R$ 0,00</span>
                </div>
            </div>

            <div class="flex flex-col sm:flex-row space-y-4 sm:space-y-0 sm:space-x-4">
                <!-- Botão para a categoria "A Receber" -->
                <div id="show-to-receive" class="category-button bg-green-200 text-green-800 font-bold p-4 rounded-lg shadow-md hover:bg-green-300 flex-1 text-center text-lg">
                    A Receber
                </div>
                <!-- Botão para a categoria "A Pagar" -->
                <div id="show-to-pay" class="category-button bg-red-200 text-red-800 font-bold p-4 rounded-lg shadow-md hover:bg-red-300 flex-1 text-center text-lg">
                    A Pagar
                </div>
            </div>

            <!-- Botão para adicionar nova pessoa -->
            <button id="add-person-button"
                    class="w-full bg-blue-600 text-white font-bold py-3 px-6 rounded-lg shadow-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 transition-colors mt-4">
                Adicionar Nova Pessoa
            </button>
        </div>

        <!-- Seção de Lista de Pessoas por Categoria (Tela 2) -->
        <div id="category-list-view" class="hidden space-y-6">
            <div class="flex items-center mb-4">
                <button id="back-to-summary-button" class="bg-gray-200 text-gray-800 font-bold py-2 px-4 rounded-lg shadow-md hover:bg-gray-300 focus:outline-none focus:ring-2 focus:ring-gray-400 transition-colors mr-4">
                    ← Voltar
                </button>
                <h2 id="category-list-title" class="text-2xl font-bold text-gray-700"></h2>
            </div>
            
            <div id="person-list-container" class="space-y-3">
                <!-- A lista de pessoas será adicionada aqui via JavaScript -->
            </div>
            
            <button id="add-transaction-from-list-button"
                    class="w-full bg-blue-600 text-white font-bold py-3 px-6 rounded-lg shadow-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 transition-colors mt-4">
                Adicionar Transação
            </button>
        </div>
        
        <!-- Seção do formulário de transação (Tela 3) -->
        <div id="transaction-form-view" class="hidden space-y-6">
            <button id="back-to-list-from-form-button" class="bg-gray-200 text-gray-800 font-bold py-2 px-4 rounded-lg shadow-md hover:bg-gray-300 focus:outline-none focus:ring-2 focus:ring-gray-400 transition-colors mb-4">
                ← Voltar
            </button>
            <h2 class="text-2xl font-bold text-gray-700 mb-4">Nova Transação</h2>
            <div class="bg-gray-50 p-4 rounded-lg border border-gray-200 space-y-4">
                <!-- Selecionar Pessoa -->
                <div>
                    <label for="person-select-transaction" class="block text-sm font-semibold text-gray-700 mb-1">Pessoa:</label>
                    <select id="person-select-transaction" class="w-full p-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500">
                        <!-- Opções serão preenchidas via JS -->
                    </select>
                </div>
                <!-- Tipo de transação (empréstimo/pagamento) -->
                <div>
                    <label for="type-select" class="block text-sm font-semibold text-gray-700 mb-1">Tipo:</label>
                    <select id="type-select" class="w-full p-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500">
                        <option value="loan">Empréstimo</option>
                        <option value="payment">Pagamento</option>
                    </select>
                </div>
                <!-- Valor da transação -->
                <div>
                    <label for="amount-input" class="block text-sm font-semibold text-gray-700 mb-1">Valor:</label>
                    <input type="number" id="amount-input" placeholder="R$ 0,00"
                           class="w-full p-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500">
                </div>
                <!-- Data da transação -->
                <div>
                    <label for="date-input" class="block text-sm font-semibold text-gray-700 mb-1">Data da Transação:</label>
                    <input type="date" id="date-input"
                           class="w-full p-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500">
                </div>
                <!-- Descrição -->
                <div>
                    <label for="description-input" class="block text-sm font-semibold text-gray-700 mb-1">Descrição:</label>
                    <input type="text" id="description-input" placeholder="Ex: Parcela 1, Lanche da noite"
                           class="w-full p-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500">
                </div>
                <!-- Forma de pagamento/empréstimo -->
                <div>
                    <label for="method-input" class="block text-sm font-semibold text-gray-700 mb-1">Forma:</label>
                    <input type="text" id="method-input" placeholder="Ex: Dinheiro, PIX, Cartão"
                           class="w-full p-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500">
                </div>
                <!-- Botão para adicionar a transação -->
                <button id="add-transaction-button"
                        class="w-full bg-blue-600 text-white font-bold py-3 px-6 rounded-lg shadow-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 transition-colors">
                    Adicionar Transação
                </button>
            </div>
        </div>

        <!-- Seção de Detalhes da Pessoa (Tela 4) -->
        <div id="details-view" class="hidden space-y-6">
            <div class="flex items-center justify-between mb-4">
                <button id="back-to-list-button" class="bg-gray-200 text-gray-800 font-bold py-2 px-4 rounded-lg shadow-md hover:bg-gray-300 focus:outline-none focus:ring-2 focus:ring-gray-400 transition-colors">
                    ← Voltar
                </button>
            </div>
            
            <!-- Saldo e Botões de Ação na tela de detalhes -->
            <div class="bg-gray-50 p-4 rounded-lg border border-gray-200">
                <p class="text-xl font-bold mb-2">
                    Saldo de <span id="person-name-display" class="text-blue-600"></span>: <span id="balance-display" class="font-extrabold text-green-600">R$ 0,00</span>
                </p>
                <div class="flex flex-col sm:flex-row space-y-2 sm:space-y-0 sm:space-x-2 mt-4">
                    <button id="generate-pdf-button"
                            class="flex-1 bg-gray-700 text-white font-bold py-2 px-4 rounded-lg shadow-md hover:bg-gray-800 focus:outline-none focus:ring-2 focus:ring-gray-600 transition-colors">
                        Gerar Extrato (PDF)
                    </button>
                    <button id="share-whatsapp-button"
                            class="flex-1 bg-green-500 text-white font-bold py-2 px-4 rounded-lg shadow-md hover:bg-green-600 focus:outline-none focus:ring-2 focus:ring-green-400 transition-colors">
                        Compartilhar no WhatsApp
                    </button>
                </div>
            </div>

            <!-- Seção do histórico de transações -->
            <div>
                <h2 class="text-2xl font-bold text-gray-700 mb-4 border-b-2 border-gray-200 pb-2">
                    Histórico de Transações
                </h2>
                <ul id="transaction-list" class="space-y-4 transaction-list">
                    <!-- As transações serão inseridas aqui dinamicamente pelo JavaScript -->
                </ul>
            </div>
        </div>
    </div>
    
    <!-- Mensagem de erro/aviso temporária -->
    <div id="message-box" class="fixed bottom-4 left-1/2 -translate-x-1/2 bg-red-500 text-white p-3 rounded-lg shadow-lg hidden"></div>

    <script>
        // Chave usada para salvar e carregar os dados no Local Storage
        const STORAGE_KEY = 'loanTrackerData';

        // Dados do aplicativo, armazenados como um objeto com duas categorias principais
        // Cada pessoa agora é um objeto com 'contact' e 'transactions'
        let appData = {
            toReceive: {},
            toPay: {}
        };

        // Mapeamento dos tipos de transação para texto amigável
        const transactionTypeMap = {
            'loan': 'Empréstimo',
            'payment': 'Pagamento'
        };

        // Variáveis de estado para controlar a visualização e a pessoa selecionada
        let currentCategory = '';
        let selectedPerson = '';

        // Referências aos elementos HTML
        const summaryView = document.getElementById('summary-view');
        const categoryListView = document.getElementById('category-list-view');
        const detailsView = document.getElementById('details-view');
        const transactionFormView = document.getElementById('transaction-form-view');

        const toReceiveBalanceDisplay = document.getElementById('to-receive-balance');
        const toPayBalanceDisplay = document.getElementById('to-pay-balance');
        const showToReceiveButton = document.getElementById('show-to-receive');
        const showToPayButton = document.getElementById('show-to-pay');
        const addPersonButton = document.getElementById('add-person-button');

        const backToSummaryButton = document.getElementById('back-to-summary-button');
        const backToListButton = document.getElementById('back-to-list-button');
        const backToListFromFormButton = document.getElementById('back-to-list-from-form-button');

        const categoryListTitle = document.getElementById('category-list-title');
        const personListContainer = document.getElementById('person-list-container');
        const addTransactionFromListButton = document.getElementById('add-transaction-from-list-button');

        const personNameDisplay = document.getElementById('person-name-display');
        const balanceDisplay = document.getElementById('balance-display');
        const generatePdfButton = document.getElementById('generate-pdf-button');
        const shareWhatsappButton = document.getElementById('share-whatsapp-button');

        const personSelectTransaction = document.getElementById('person-select-transaction');
        const typeSelect = document.getElementById('type-select');
        const amountInput = document.getElementById('amount-input');
        const dateInput = document.getElementById('date-input');
        const descriptionInput = document.getElementById('description-input');
        const methodInput = document.getElementById('method-input');
        const addTransactionButton = document.getElementById('add-transaction-button');
        const transactionList = document.getElementById('transaction-list');
        const messageBox = document.getElementById('message-box');

        // --- Funções para carregar e salvar os dados ---

        /**
         * Exibe uma mensagem de erro ou aviso na tela por alguns segundos.
         * @param {string} message - A mensagem a ser exibida.
         * @param {string} type - O tipo de mensagem ('success' ou 'error').
         */
        function showMessage(message, type = 'error') {
            messageBox.textContent = message;
            messageBox.classList.remove('hidden', 'bg-red-500', 'bg-green-500');
            messageBox.classList.add(type === 'success' ? 'bg-green-500' : 'bg-red-500');
            messageBox.classList.remove('hidden');
            setTimeout(() => {
                messageBox.classList.add('hidden');
            }, 3000);
        }

        /**
         * Carrega os dados do Local Storage e preenche o `appData`.
         */
        function loadData() {
            try {
                const storedData = localStorage.getItem(STORAGE_KEY);
                if (storedData) {
                    const parsedData = JSON.parse(storedData);
                    // Garante que a estrutura de dados seja sempre a mesma, incluindo o novo formato
                    appData.toReceive = parsedData.toReceive || {};
                    appData.toPay = parsedData.toPay || {};

                    // Atualiza a estrutura de dados de versões anteriores, se necessário
                    Object.keys(appData.toReceive).forEach(key => {
                        if (!appData.toReceive[key].transactions) {
                            appData.toReceive[key] = {
                                contact: '',
                                transactions: appData.toReceive[key]
                            };
                        }
                    });
                    Object.keys(appData.toPay).forEach(key => {
                        if (!appData.toPay[key].transactions) {
                            appData.toPay[key] = {
                                contact: '',
                                transactions: appData.toPay[key]
                            };
                        }
                    });

                }
            } catch (e) {
                console.error("Erro ao carregar os dados do Local Storage:", e);
                appData = { toReceive: {}, toPay: {} };
            }
        }

        /**
         * Salva o `appData` no Local Storage.
         */
        function saveData() {
            try {
                localStorage.setItem(STORAGE_KEY, JSON.stringify(appData));
            } catch (e) {
                console.error("Erro ao salvar os dados no Local Storage:", e);
            }
        }

        // --- Funções de renderização ---

        /**
         * Calcula e exibe os saldos globais (a pagar e a receber).
         */
        function renderGlobalBalances() {
            let totalToReceive = 0;
            let totalToPay = 0;
            
            // Calcula o saldo total a receber
            Object.values(appData.toReceive).forEach(person => {
                const balance = person.transactions.reduce((acc, t) => acc + (t.type === 'payment' ? t.amount : -t.amount), 0);
                totalToReceive += balance;
            });
            
            // Calcula o saldo total a pagar
            Object.values(appData.toPay).forEach(person => {
                const balance = person.transactions.reduce((acc, t) => acc + (t.type === 'payment' ? t.amount : -t.amount), 0);
                totalToPay += balance;
            });

            toReceiveBalanceDisplay.textContent = new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(totalToReceive);
            toPayBalanceDisplay.textContent = new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(totalToPay);
        }

        /**
         * Renderiza a lista de pessoas para a categoria selecionada.
         * @param {string} category - A categoria ('toReceive' ou 'toPay').
         */
        function renderPersonList(category) {
            personListContainer.innerHTML = '';
            const people = appData[category];
            const sortedNames = Object.keys(people).sort();

            if (sortedNames.length === 0) {
                personListContainer.innerHTML = `<p class="text-center text-gray-500">Nenhum nome cadastrado nesta categoria.</p>`;
                return;
            }

            sortedNames.forEach(personName => {
                const personItem = document.createElement('button');
                personItem.className = 'w-full text-left bg-gray-100 text-gray-700 font-semibold py-3 px-4 rounded-lg shadow hover:bg-gray-200 transition-colors person-select-button';
                personItem.textContent = personName;
                personItem.dataset.personName = personName;
                personItem.dataset.category = category;
                personListContainer.appendChild(personItem);
            });
        }

        /**
         * Preenche o seletor de pessoas na tela de transação.
         * @param {string} category - A categoria ('toReceive' ou 'toPay').
         */
        function populatePersonSelect(category) {
            personSelectTransaction.innerHTML = '';
            const people = appData[category];
            const sortedNames = Object.keys(people).sort();

            if (sortedNames.length === 0) {
                const defaultOption = document.createElement('option');
                defaultOption.textContent = 'Nenhuma pessoa cadastrada';
                defaultOption.value = '';
                personSelectTransaction.appendChild(defaultOption);
            } else {
                sortedNames.forEach(personName => {
                    const option = document.createElement('option');
                    option.value = personName;
                    option.textContent = personName;
                    personSelectTransaction.appendChild(option);
                });
            }
        }

        /**
         * Calcula e exibe o saldo de uma pessoa.
         * @param {string} personName - O nome da pessoa.
         */
        function renderBalance(personName) {
            let balance = 0;
            if (appData[currentCategory][personName]) {
                appData[currentCategory][personName].transactions.forEach(transaction => {
                    if (transaction.type === 'payment') {
                        balance += transaction.amount;
                    } else {
                        balance -= transaction.amount;
                    }
                });
            }

            const formattedBalance = new Intl.NumberFormat('pt-BR', {
                style: 'currency',
                currency: 'BRL'
            }).format(balance);

            balanceDisplay.textContent = formattedBalance;
            balanceDisplay.classList.toggle('text-red-600', balance < 0);
            balanceDisplay.classList.toggle('text-green-600', balance >= 0);
        }

        /**
         * Exibe a lista de transações de uma pessoa.
         * @param {string} personName - O nome da pessoa.
         */
        function renderTransactions(personName) {
            transactionList.innerHTML = '';
            const transactions = appData[currentCategory][personName]?.transactions;

            if (transactions && transactions.length > 0) {
                // Itera sobre as transações da pessoa, da mais recente para a mais antiga
                transactions.slice().reverse().forEach((transaction, index) => {
                    // O índice original é usado para deletar a transação correta
                    const originalIndex = transactions.length - 1 - index;
                    const listItem = document.createElement('li');
                    
                    listItem.className = `transaction-item ${transaction.type} bg-white p-4 rounded-lg shadow-sm flex justify-between items-center`;
                    
                    const amountClass = transaction.type === 'loan' ? 'text-red-500' : 'text-green-500';
                    const formattedAmount = `R$ ${transaction.amount.toFixed(2).replace('.', ',')}`;

                    // Adiciona a descrição se ela existir
                    const descriptionHtml = transaction.description ? `<p class="text-sm text-gray-500 italic">${transaction.description}</p>` : '';
                    const dateDisplay = transaction.date ? new Date(transaction.date + 'T12:00:00').toLocaleDateString('pt-BR') : 'Data não informada';

                    listItem.innerHTML = `
                        <div class="flex-1">
                            <p class="font-bold text-gray-800">${transactionTypeMap[transaction.type]} - ${transaction.method}</p>
                            ${descriptionHtml}
                            <p class="text-xs text-gray-400 mt-1">Data: ${dateDisplay}</p>
                        </div>
                        <p class="font-extrabold ${amountClass} mr-4">${formattedAmount}</p>
                        <!-- Botão de exclusão -->
                        <button class="delete-button text-gray-400 hover:text-red-500 transition-colors" data-index="${originalIndex}">
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                                <path fill-rule="evenodd" d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm6 0a1 1 0 012 0v6a1 1 0 11-2 0V8z" clip-rule="evenodd" />
                            </svg>
                        </button>
                    `;
                    transactionList.appendChild(listItem);
                });
            } else {
                transactionList.innerHTML = `<p class="text-center text-gray-500">Nenhuma transação registrada.</p>`;
            }
        }

        // --- Funções de manipulação de dados ---

        /**
         * Adiciona uma nova pessoa ao `appData` e atualiza a interface.
         */
        function addPerson() {
            const personName = prompt("Digite o nome da nova pessoa:").trim();
            if (!personName) {
                showMessage('Por favor, digite um nome para a pessoa.', 'error');
                return;
            }

            const personType = prompt(`Digite a categoria para "${personName}":\n\n1. "A Receber" (digite 'receber')\n2. "A Pagar" (digite 'pagar')`).toLowerCase();
            let categoryKey = '';

            if (personType === 'receber') {
                categoryKey = 'toReceive';
            } else if (personType === 'pagar') {
                categoryKey = 'toPay';
            } else {
                showMessage('Tipo de categoria inválido. Por favor, digite "receber" ou "pagar".', 'error');
                return;
            }

            if (appData[categoryKey][personName]) {
                showMessage("Essa pessoa já existe nessa categoria.", 'error');
            } else {
                const contact = prompt(`Digite o número de telefone de ${personName} (ex: 5541999998888):`).trim();
                appData[categoryKey][personName] = { contact: contact, transactions: [] };
                saveData();
                renderGlobalBalances();
                showMessage(`Pessoa "${personName}" adicionada com sucesso!`, 'success');
            }
        }

        /**
         * Exclui uma transação específica e atualiza a interface.
         * @param {string} personName - O nome da pessoa.
         * @param {number} transactionIndex - O índice da transação a ser excluída.
         */
        function deleteTransaction(personName, transactionIndex) {
            if (appData[currentCategory][personName]?.transactions && appData[currentCategory][personName].transactions[transactionIndex]) {
                appData[currentCategory][personName].transactions.splice(transactionIndex, 1);
                saveData();
                renderBalance(personName);
                renderTransactions(personName);
                renderGlobalBalances(); // Atualiza o saldo global também
                showMessage('Transação excluída com sucesso!', 'success');
            }
        }

        /**
         * Gera um documento PDF com o histórico de transações da pessoa selecionada.
         */
        function generatePDFStatement() {
            if (!selectedPerson) {
                showMessage("Por favor, selecione uma pessoa para gerar o extrato.", 'error');
                return;
            }

            const transactions = appData[currentCategory][selectedPerson]?.transactions;
            if (!transactions || transactions.length === 0) {
                showMessage("Não há transações para gerar o extrato.", 'error');
                return;
            }
            
            // Cria um novo documento PDF
            const { jsPDF } = window.jspdf;
            const doc = new jsPDF();

            // Título
            doc.setFontSize(22);
            doc.text(`Extrato de Empréstimos: ${selectedPerson}`, 10, 20);

            // Saldo Atual
            doc.setFontSize(14);
            const balance = transactions.reduce((acc, t) => acc + (t.type === 'payment' ? t.amount : -t.amount), 0);
            const formattedBalance = new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(balance);
            doc.text(`Saldo Atual: ${formattedBalance}`, 10, 30);

            // Tabela de transações
            const headers = [['Data', 'Tipo', 'Valor', 'Forma', 'Descrição']];
            const data = transactions.map(t => [
                new Date(t.date + 'T12:00:00').toLocaleDateString('pt-BR'),
                transactionTypeMap[t.type],
                `R$ ${t.amount.toFixed(2).replace('.', ',')}`,
                t.method,
                t.description || ''
            ]);
            
            // Usa autoTable para gerar uma tabela formatada
            doc.autoTable({
                startY: 40,
                head: headers,
                body: data,
                headStyles: { fillColor: [52, 73, 94] },
                alternateRowStyles: { fillColor: [245, 245, 245] },
                styles: { font: 'helvetica' },
            });

            // Salva o documento com um nome de arquivo
            doc.save(`extrato-${selectedPerson.toLowerCase().replace(/\s/g, '-')}.pdf`);
            showMessage("Extrato PDF gerado!", 'success');
        }

        // --- Lógica de eventos ---

        // Navegação entre as telas
        showToReceiveButton.addEventListener('click', () => {
            currentCategory = 'toReceive';
            categoryListTitle.textContent = 'Pessoas para Receber';
            summaryView.classList.add('hidden');
            categoryListView.classList.remove('hidden');
            renderPersonList(currentCategory);
        });

        showToPayButton.addEventListener('click', () => {
            currentCategory = 'toPay';
            categoryListTitle.textContent = 'Pessoas para Pagar';
            summaryView.classList.add('hidden');
            categoryListView.classList.remove('hidden');
            renderPersonList(currentCategory);
        });

        addTransactionFromListButton.addEventListener('click', () => {
            if (Object.keys(appData[currentCategory]).length === 0) {
                showMessage("Por favor, adicione uma pessoa primeiro.", 'error');
                return;
            }
            categoryListView.classList.add('hidden');
            transactionFormView.classList.remove('hidden');
            populatePersonSelect(currentCategory);
        });

        backToSummaryButton.addEventListener('click', () => {
            summaryView.classList.remove('hidden');
            categoryListView.classList.add('hidden');
        });

        backToListButton.addEventListener('click', () => {
            detailsView.classList.add('hidden');
            categoryListView.classList.remove('hidden');
        });

        backToListFromFormButton.addEventListener('click', () => {
            transactionFormView.classList.add('hidden');
            categoryListView.classList.remove('hidden');
        });

        // Adiciona um evento de clique para o container da lista de pessoas
        personListContainer.addEventListener('click', (event) => {
            const personButton = event.target.closest('.person-select-button');
            if (personButton) {
                selectedPerson = personButton.dataset.personName;
                currentCategory = personButton.dataset.category;
                
                categoryListView.classList.add('hidden');
                detailsView.classList.remove('hidden');

                personNameDisplay.textContent = selectedPerson;
                renderBalance(selectedPerson);
                renderTransactions(selectedPerson);
            }
        });

        // Ouve o clique do botão de adicionar pessoa
        addPersonButton.addEventListener('click', addPerson);

        // Ouve o clique no botão de adicionar transação
        addTransactionButton.addEventListener('click', () => {
            const selectedPersonForTransaction = personSelectTransaction.value;
            const amount = parseFloat(amountInput.value);
            const date = dateInput.value || new Date().toISOString().slice(0, 10);
            const description = descriptionInput.value.trim();
            const method = methodInput.value.trim();

            if (!selectedPersonForTransaction) {
                showMessage('Por favor, selecione uma pessoa.');
                return;
            }
            if (isNaN(amount) || amount <= 0) {
                showMessage('Por favor, insira um valor válido para a transação.');
                return;
            }
            if (!method) {
                showMessage('Por favor, insira a forma de pagamento/empréstimo.');
                return;
            }

            const newTransaction = {
                type: typeSelect.value,
                amount: amount,
                date: date,
                description: description,
                method: method
            };

            if (appData[currentCategory][selectedPersonForTransaction]) {
                appData[currentCategory][selectedPersonForTransaction].transactions.push(newTransaction);
            } else {
                showMessage('Erro ao adicionar transação. Pessoa não encontrada.', 'error');
                return;
            }

            saveData();
            renderGlobalBalances(); // Atualiza o saldo global também

            amountInput.value = '';
            dateInput.value = '';
            descriptionInput.value = '';
            methodInput.value = '';
            showMessage('Transação adicionada com sucesso!', 'success');
        });

        // Ouve o clique no botão para gerar PDF
        generatePdfButton.addEventListener('click', generatePDFStatement);
        
        // Ouve o clique no botão de compartilhar no WhatsApp
        shareWhatsappButton.addEventListener('click', () => {
            if (!selectedPerson) {
                showMessage('Selecione uma pessoa primeiro.', 'error');
                return;
            }
            const personData = appData[currentCategory][selectedPerson];
            const contact = personData.contact.replace(/\D/g, ''); // Remove caracteres não numéricos
            
            if (!contact) {
                showMessage('Número de WhatsApp não cadastrado para esta pessoa.', 'error');
                return;
            }

            // Apenas gera o PDF, o usuário precisa anexá-lo manualmente no WhatsApp
            generatePDFStatement();
            
            const whatsappMessage = `Olá, ${selectedPerson}! Segue o extrato atualizado das nossas contas.`;
            const whatsappUrl = `https://api.whatsapp.com/send?phone=${contact}&text=${encodeURIComponent(whatsappMessage)}`;
            
            window.open(whatsappUrl, '_blank');
        });
        
        // Ouve os cliques nos botões de exclusão (delegação de evento)
        transactionList.addEventListener('click', (event) => {
            const deleteButton = event.target.closest('.delete-button');
            if (deleteButton) {
                const transactionIndex = parseInt(deleteButton.dataset.index);
                if (selectedPerson) {
                    deleteTransaction(selectedPerson, transactionIndex);
                }
            }
        });

        // --- Inicialização do aplicativo ---
        
        // Primeiro, tenta carregar os dados salvos
        loadData();
        // E renderiza o saldo global inicial
        renderGlobalBalances();
    </script>
</body>
</html>
