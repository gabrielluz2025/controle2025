<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Controle Financeiro Pessoal Completo</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/xlsx/dist/xlsx.full.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        /* Define a fonte Inter para toda a página */
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f4f8; /* Cor de fundo suave */
        }

        /* Estilos para a mensagem de feedback */
        #feedback-message {
            display: none;
            padding: 12px;
            margin-top: 16px;
            border-radius: 8px;
            font-weight: 500;
        }

        .feedback-success {
            background-color: #d1fae5;
            color: #065f46;
        }

        .feedback-error {
            background-color: #fee2e2;
            color: #991b1b;
        }

        .feedback-info {
            background-color: #e0f2fe;
            color: #075985;
        }

        /* Estilos para ocultar/mostrar elementos */
        .hidden {
            display: none !important;
        }

        /* Estilos para a seção colapsável */
        .collapsible-header {
            cursor: pointer;
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 1rem 0;
            border-bottom: 1px solid #e2e8f0;
            margin-bottom: 1rem;
        }

        .collapsible-header:hover {
            background-color: #f8fafc;
        }

        .collapsible-header h2 {
            margin-bottom: 0;
        }

        .arrow-icon {
            transition: transform 0.3s ease;
        }

        .arrow-icon.rotated {
            transform: rotate(90deg);
        }

        /* Estilo para transações pagas */
        .paid-transaction {
            text-decoration: line-through;
            color: #6b7280; /* gray-500 */
        }
    </style>
</head>
<body class="p-4 sm:p-6 md:p-8 lg:p-10">
    <div class="max-w-5xl mx-auto bg-white rounded-xl shadow-lg p-6 sm:p-8">
        <h1 class="text-3xl sm:text-4xl font-bold text-center text-gray-800 mb-8">Controle Financeiro Pessoal Completo</h1>

        <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-8">
            <div class="bg-blue-100 p-4 rounded-lg shadow-md">
                <h2 class="text-lg font-semibold text-blue-800">Total de Receitas</h2>
                <p id="total-receitas" class="text-2xl font-bold text-blue-700">R$ 0,00</p>
            </div>
            <div class="bg-red-100 p-4 rounded-lg shadow-md">
                <h2 class="text-lg font-semibold text-red-800">Total de Despesas (Mês Atual)</h2>
                <p id="total-despesas" class="text-2xl font-bold text-red-700">R$ 0,00</p>
            </div>
            <div class="bg-green-100 p-4 rounded-lg shadow-md">
                <h2 class="text-lg font-semibold text-green-800">Saldo Atual</h2>
                <p id="saldo-atual" class="text-2xl font-bold text-green-700">R$ 0,00</p>
            </div>
        </div>
        <div class="bg-purple-100 p-4 rounded-lg shadow-md mb-8">
            <h2 class="text-lg font-semibold text-purple-800">Total de Faturas de Cartão (Mês Atual)</h2>
            <p id="total-card-bills" class="text-2xl font-bold text-purple-700">R$ 0,00</p>
        </div>

        <div class="mb-8 border-b pb-6 border-gray-200">
            <div id="detailed-card-bills-header" class="collapsible-header">
                <h2 class="text-2xl font-semibold text-gray-700">Faturas Detalhadas de Cartão</h2>
                <svg id="detailed-card-bills-arrow" class="arrow-icon w-6 h-6 text-gray-600" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"></path>
                </svg>
            </div>
            <div id="detailed-card-bills-content" class="hidden">
                <ul id="detailed-card-bills-list" class="bg-white rounded-md shadow-sm divide-y divide-gray-200">
                    </ul>
                <p id="no-card-bills-message" class="text-center text-gray-500 mt-4 hidden">Nenhuma fatura de cartão para exibir.</p>
            </div>
        </div>


        <div class="mb-8 border-b pb-6 border-gray-200">
            <h2 class="text-2xl font-semibold text-gray-700 mb-4">Previsão de Despesas Próximo Mês</h2>
            <div class="bg-yellow-50 p-4 rounded-lg shadow-md flex justify-between items-center">
                <h3 class="text-lg font-semibold text-yellow-800">Total Previsto:</h3>
                <p id="predicted-expenses" class="text-2xl font-bold text-yellow-700">R$ 0,00</p>
            </div>
        </div>

        <div class="mb-8 border-b pb-6 border-gray-200">
            <h2 class="text-2xl font-semibold text-gray-700 mb-4">Resumo Mensal</h2>
            <div class="flex items-center space-x-4 mb-4">
                <button id="prev-month-btn" class="px-4 py-2 bg-gray-200 rounded-md hover:bg-gray-300">Anterior</button>
                <span id="current-month-display" class="text-lg font-semibold text-gray-800"></span>
                <button id="next-month-btn" class="px-4 py-2 bg-gray-200 rounded-md hover:bg-gray-300">Próximo</button>
            </div>
            <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                <div class="bg-blue-50 p-4 rounded-lg shadow-sm">
                    <h3 class="text-lg font-semibold text-blue-700">Receitas do Mês</h3>
                    <p id="monthly-income" class="text-xl font-bold text-blue-600">R$ 0,00</p>
                </div>
                <div class="bg-red-50 p-4 rounded-lg shadow-sm">
                    <h3 class="text-lg font-semibold text-red-700">Despesas do Mês</h3>
                    <p id="monthly-expenses" class="text-xl font-bold text-red-600">R$ 0,00</p>
                </div>
                <div class="bg-green-50 p-4 rounded-lg shadow-sm">
                    <h3 class="text-lg font-semibold text-green-700">Saldo do Mês</h3>
                    <p id="monthly-balance" class="text-xl font-bold text-green-600">R$ 0,00</p>
                </div>
            </div>
        </div>

        <div class="mb-8 border-b pb-6 border-gray-200">
            <h2 class="text-2xl font-semibold text-gray-700 mb-4">Gerenciar Contas Bancárias / Dinheiro</h2>

            <div class="p-4 bg-gray-50 rounded-lg shadow-sm mb-6">
                <h3 class="text-xl font-semibold text-gray-700 mb-3">Adicionar Conta</h3>
                <form id="add-account-form" class="flex flex-col sm:flex-row gap-3">
                    <input type="text" id="account-name" placeholder="Nome da Conta (ex: Banco X, Dinheiro)" required
                           class="flex-grow px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm">
                    <select id="account-type"
                            class="px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm">
                        <option value="bank">Conta Bancária</option>
                        <option value="cash">Dinheiro</option>
                    </select>
                    <input type="number" id="initial-balance" step="0.01" placeholder="Saldo Inicial (R$)" required
                           class="w-32 px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm">
                    <input type="number" id="overdraft-limit" step="0.01" placeholder="Cheque Especial (R$)"
                           class="w-32 px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm hidden">
                    <button type="submit"
                            class="px-5 py-2 bg-blue-500 text-white font-semibold rounded-md shadow-md hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-400 transition duration-150 ease-in-out">
                        Adicionar
                    </button>
                </form>
            </div>

            <div>
                <h3 class="text-lg font-semibold text-gray-700 mb-2">Minhas Contas</h3>
                <ul id="accounts-list" class="bg-white rounded-md shadow-sm divide-y divide-gray-200">
                    </ul>
            </div>
        </div>

        <div class="mb-8 border-b pb-6 border-gray-200">
            <h2 class="text-2xl font-semibold text-gray-700 mb-4">Gerenciar Cartões de Crédito</h2>

            <div class="p-4 bg-gray-50 rounded-lg shadow-sm mb-6">
                <h3 class="text-xl font-semibold text-gray-700 mb-3">Adicionar Cartão de Crédito</h3>
                <form id="add-credit-card-form" class="flex flex-col sm:flex-row gap-3">
                    <input type="text" id="card-name" placeholder="Nome do Cartão (ex: Visa, Master)" required
                           class="flex-grow px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-purple-500 focus:border-purple-500 sm:text-sm">
                    <input type="number" id="card-limit" step="0.01" placeholder="Limite (R$)" required
                           class="w-28 px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-purple-500 focus:border-purple-500 sm:text-sm">
                    <input type="number" id="card-due-day" min="1" max="31" placeholder="Dia Venc. Fatura" required
                           class="w-32 px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-purple-500 focus:border-purple-500 sm:text-sm">
                    <button type="submit"
                            class="px-5 py-2 bg-purple-500 text-white font-semibold rounded-md shadow-md hover:bg-purple-600 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-purple-400 transition duration-150 ease-in-out">
                        Adicionar
                    </button>
                </form>
            </div>

            <div>
                <h3 class="text-lg font-semibold text-gray-700 mb-2">Meus Cartões</h3>
                <ul id="credit-cards-list" class="bg-white rounded-md shadow-sm divide-y divide-gray-200">
                    </ul>
            </div>
        </div>

        <div class="mb-8 border-b pb-6 border-gray-200">
            <h2 class="text-2xl font-semibold text-gray-700 mb-4">Adicionar Nova Transação Manualmente</h2>
            <form id="transaction-form" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
                <div>
                    <label for="date" class="block text-sm font-medium text-gray-700 mb-1">Data</label>
                    <input type="date" id="date" required
                           class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm">
                </div>
                <div>
                    <label for="description" class="block text-sm font-medium text-gray-700 mb-1">Descrição</label>
                    <input type="text" id="description" placeholder="Ex: Salário, Aluguel, Supermercado" required
                           class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm">
                </div>
                <div>
                    <label for="value" class="block text-sm font-medium text-gray-700 mb-1">Valor (R$)</label>
                    <input type="number" id="value" step="0.01" placeholder="0.00" required
                           class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm">
                </div>
                <div>
                    <label for="type" class="block text-sm font-medium text-gray-700 mb-1">Tipo</label>
                    <select id="type" required
                            class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm">
                        <option value="receita">Receita</option>
                        <option value="despesa">Despesa</option>
                    </select>
                </div>
                <div>
                    <label for="category-select" class="block text-sm font-medium text-gray-700 mb-1">Categoria</label>
                    <select id="category-select" required
                            class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm">
                        </select>
                    <input type="text" id="new-category-input" placeholder="Nova Categoria"
                           class="mt-2 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm hidden">
                </div>
                <div>
                    <label for="account-select" class="block text-sm font-medium text-gray-700 mb-1">Conta / Cartão</label>
                    <select id="account-select" required
                            class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm">
                        <option value="">Selecione uma conta</option>
                        </select>
                </div>
                <div class="col-span-1 sm:col-span-2 lg:col-span-1">
                    <label for="installments" class="block text-sm font-medium text-gray-700 mb-1">Parcelas (total)</label>
                    <input type="number" id="installments" min="1" value="1"
                           class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm">
                </div>
                <div class="col-span-1 sm:col-span-2 lg:col-span-1">
                    <label for="installments-paid" class="block text-sm font-medium text-gray-700 mb-1">Parcelas já pagas</label>
                    <input type="number" id="installments-paid" min="0" value="0"
                           class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm">
                </div>
                <div class="sm:col-span-2 lg:col-span-full flex items-end justify-end">
                    <button type="submit"
                            class="w-full sm:w-auto px-6 py-2 bg-blue-600 text-white font-semibold rounded-md shadow-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 transition duration-150 ease-in-out">
                        Adicionar Transação
                    </button>
                </div>
            </form>
        </div>

        <div class="mb-8 border-b pb-6 border-gray-200">
            <div id="import-header" class="collapsible-header">
                <h2 class="text-2xl font-semibold text-gray-700">Importar Extratos</h2>
                <svg id="import-arrow" class="arrow-icon w-6 h-6 text-gray-600" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"></path>
                </svg>
            </div>
            
            <div id="import-section-content" class="hidden">
                <p class="text-sm text-gray-600 mb-4">
                    Seu arquivo deve ser OFX, CSV ou Excel e conter informações de transação com Data, Descrição, Valor e Tipo.
                    <span class="font-bold text-red-600">Atenção: Integração direta com bancos (Open Finance) não é possível em uma aplicação web no navegador por motivos de segurança e privacidade.</span>
                    <span class="font-bold text-red-600">Importação por imagem não é suportada (requer OCR e processamento de servidor).</span>
                </p>

                <div class="p-4 bg-gray-50 rounded-lg shadow-sm">
                    <h3 class="text-xl font-semibold text-gray-700 mb-3">Carregar Arquivo de Extrato</h3>
                    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4 items-end">
                        <div>
                            <label for="file-input" class="block text-sm font-medium text-gray-700 mb-1">Selecione o Arquivo</label>
                            <input type="file" id="file-input" accept=".ofx, .csv, .xls, .xlsx"
                                   class="block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-md file:border-0 file:text-sm file:font-semibold file:bg-gray-100 file:text-gray-700 hover:file:bg-gray-200 cursor-pointer">
                        </div>
                        <div class="flex justify-end col-span-full sm:col-span-1">
                            <button id="import-file-btn"
                                    class="w-full sm:w-auto px-6 py-2 bg-indigo-600 text-white font-semibold rounded-md shadow-md hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 transition duration-150 ease-in-out">
                                Carregar para Revisão
                            </button>
                        </div>
                    </div>
                </div>
            </div>
            <div id="feedback-message" class="hidden"></div>
        </div>

        <div id="import-review-section" class="mb-8 border-b pb-6 border-gray-200 hidden">
            <h2 class="text-2xl font-semibold text-gray-700 mb-4">Revisar Transações Importadas</h2>
            <p class="text-sm text-gray-600 mb-4">
                Vincule cada transação importada a uma de suas contas ou cartões.
            </p>
            <div class="overflow-x-auto rounded-lg shadow-md mb-4">
                <table class="min-w-full divide-y divide-gray-200">
                    <thead class="bg-yellow-50">
                        <tr>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Data</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Descrição</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Valor</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Tipo</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Categoria</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Vincular a</th>
                        </tr>
                    </thead>
                    <tbody id="imported-transactions-table-body" class="bg-white divide-y divide-gray-200">
                        </tbody>
                </table>
            </div>
            <div class="flex justify-end gap-4">
                <button id="cancel-review-btn"
                        class="px-6 py-2 bg-gray-400 text-white font-semibold rounded-md shadow-md hover:bg-gray-500 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-300 transition duration-150 ease-in-out">
                    Cancelar Importação
                </button>
                <button id="confirm-import-btn"
                        class="px-6 py-2 bg-blue-600 text-white font-semibold rounded-md shadow-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 transition duration-150 ease-in-out">
                    Confirmar Importação
                </button>
            </div>
        </div>

        <div class="mb-8 border-b pb-6 border-gray-200">
            <h2 class="text-2xl font-semibold text-gray-700 mb-4">Dashboard de Visualização (Despesas por Categoria)</h2>
            <div class="bg-white p-4 rounded-lg shadow-md">
                <canvas id="expenses-chart"></canvas>
            </div>
        </div>

        <div class="mb-8 border-b pb-6 border-gray-200">
            <h2 class="text-2xl font-semibold text-gray-700 mb-4">Baixar Planilha</h2>
            <button id="download-xlsx-btn"
                    class="w-full sm:w-auto px-6 py-2 bg-gray-600 text-white font-semibold rounded-md shadow-md hover:bg-gray-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500 transition duration-150 ease-in-out">
                Baixar Transações do Mês como Excel (.xlsx)
            </button>
        </div>

        <div>
            <h2 class="text-2xl font-semibold text-gray-700 mb-4">Minhas Transações</h2>
            <div class="overflow-x-auto rounded-lg shadow-md">
                <table class="min-w-full divide-y divide-gray-200">
                    <thead class="bg-gray-50">
                        <tr>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Data</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Descrição</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Valor</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Tipo</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Categoria</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Conta / Cartão</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Parcela</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Ações</th>
                        </tr>
                    </thead>
                    <tbody id="transaction-list" class="bg-white divide-y divide-gray-200">
                        </tbody>
                </table>
            </div>
            <p id="no-transactions-message" class="text-center text-gray-500 mt-4 hidden">Nenhuma transação registrada ainda.</p>
        </div>

    </div>

    <script>
        // Arrays globais para armazenar dados
        let accounts = [];
        let creditCards = [];
        let transactions = [];
        let categories = [
            "Alimentação", "Transporte", "Moradia", "Lazer", "Saúde", "Educação",
            "Salário", "Investimentos", "Contas Fixas", "Compras", "Outros"
        ];
        // Array temporário para transações importadas pendentes de revisão
        let importedTransactionsToReview = [];
        // Mês e ano atuais para o resumo mensal e filtro de transações
        let currentMonth = new Date().getMonth(); // 0-indexed (Jan = 0)
        let currentYear = new Date().getFullYear();

        // Instância do Chart.js para o gráfico de despesas
        let expensesChart = null;

        // Referências aos elementos do DOM
        const totalReceitasEl = document.getElementById('total-receitas');
        const totalDespesasEl = document.getElementById('total-despesas');
        const saldoAtualEl = document.getElementById('saldo-atual');
        const totalCardBillsEl = document.getElementById('total-card-bills');
        const predictedExpensesEl = document.getElementById('predicted-expenses');
        const noTransactionsMessage = document.getElementById('no-transactions-message');
        const feedbackMessageEl = document.getElementById('feedback-message');

        // Gerenciamento de Contas e Cartões
        const addAccountForm = document.getElementById('add-account-form');
        const accountsList = document.getElementById('accounts-list');
        const addCreditCardForm = document.getElementById('add-credit-card-form');
        const creditCardsList = document.getElementById('credit-cards-list');
        const accountSelect = document.getElementById('account-select'); // Para transações manuais
        const accountTypeSelect = document.getElementById('account-type'); // Para mostrar/ocultar limite de cheque especial
        const initialBalanceInput = document.getElementById('initial-balance');
        const overdraftLimitInput = document.getElementById('overdraft-limit');
        const cardDueDayInput = document.getElementById('card-due-day');

        // Formulário de Transações
        const transactionForm = document.getElementById('transaction-form');
        const transactionList = document.getElementById('transaction-list');
        const installmentsInput = document.getElementById('installments');
        const installmentsPaidInput = document.getElementById('installments-paid');
        const transactionTypeSelect = document.getElementById('type');
        const categorySelect = document.getElementById('category-select');
        const newCategoryInput = document.getElementById('new-category-input');

        // Seções de Importação
        const importHeader = document.getElementById('import-header');
        const importSectionContent = document.getElementById('import-section-content');
        const importArrow = document.getElementById('import-arrow');
        const importReviewSection = document.getElementById('import-review-section');
        const importedTransactionsTableBody = document.getElementById('imported-transactions-table-body');
        const confirmImportBtn = document.getElementById('confirm-import-btn');
        const cancelReviewBtn = document.getElementById('cancel-review-btn');

        // Importação Unificada
        const fileInput = document.getElementById('file-input');
        const importFileBtn = document.getElementById('import-file-btn');

        // Elementos do Resumo Mensal
        const prevMonthBtn = document.getElementById('prev-month-btn');
        const nextMonthBtn = document.getElementById('next-month-btn');
        const currentMonthDisplay = document.getElementById('current-month-display');
        const monthlyIncomeEl = document.getElementById('monthly-income');
        const monthlyExpensesEl = document.getElementById('monthly-expenses');
        const monthlyBalanceEl = document.getElementById('monthly-balance');

        // Download
        const downloadXlsxBtn = document.getElementById('download-xlsx-btn');

        // Elementos para Faturas Detalhadas de Cartão
        const detailedCardBillsHeader = document.getElementById('detailed-card-bills-header');
        const detailedCardBillsContent = document.getElementById('detailed-card-bills-content');
        const detailedCardBillsArrow = document.getElementById('detailed-card-bills-arrow');
        const detailedCardBillsList = document.getElementById('detailed-card-bills-list');
        const noCardBillsMessage = document.getElementById('no-card-bills-message');


        // --- Funções Utilitárias ---

        /**
         * Exibe uma mensagem de feedback para o usuário.
         * @param {string} message - A mensagem a ser exibida.
         * @param {'success'|'error'|'info'} type - O tipo de mensagem (afeta a cor).
         */
        const showFeedback = (message, type) => {
            feedbackMessageEl.textContent = message;
            feedbackMessageEl.classList.remove('hidden', 'feedback-success', 'feedback-error', 'feedback-info');
            feedbackMessageEl.classList.add(`feedback-${type}`);
            feedbackMessageEl.style.display = 'block'; // Garante que esteja visível

            setTimeout(() => {
                feedbackMessageEl.style.display = 'none'; // Oculta após 5 segundos
            }, 5000);
        };

        /**
         * Formata um valor numérico para o formato de moeda BRL.
         * @param {number} value - O valor a ser formatado.
         * @returns {string} O valor formatado como moeda.
         */
        const formatCurrency = (value) => {
            return new Intl.NumberFormat('pt-BR', {
                style: 'currency',
                currency: 'BRL'
            }).format(value);
        };

        /**
         * Gera um ID único simples.
         * @returns {string} Um ID único.
         */
        const generateUniqueId = () => '_' + Math.random().toString(36).substr(2, 9);

        // --- Persistência de Dados (localStorage) ---

        /**
         * Carrega todos os dados do localStorage ao iniciar a aplicação.
         */
        const loadData = () => {
            const storedAccounts = localStorage.getItem('financialAccounts');
            if (storedAccounts) {
                accounts = JSON.parse(storedAccounts);
            }
            const storedCreditCards = localStorage.getItem('financialCreditCards');
            if (storedCreditCards) {
                creditCards = JSON.parse(storedCreditCards);
            }
            const storedTransactions = localStorage.getItem('financialTransactions');
            if (storedTransactions) {
                transactions = JSON.parse(storedTransactions);
            }
            const storedCategories = localStorage.getItem('financialCategories');
            if (storedCategories) {
                categories = JSON.parse(storedCategories);
            }

            // Renderiza e atualiza a UI com os dados carregados
            renderAccounts();
            renderCreditCards();
            populateAccountSelects();
            populateCategorySelect();
            renderTransactions();
            updateSummary();
            calculateNextMonthExpensesPrediction();
            renderMonthlySummary();
            renderExpensesChart();
            renderDetailedCardBills();
            checkCreditCardDueDates(); // Verifica datas de vencimento ao carregar
        };

        /** Salva as contas no localStorage. */
        const saveAccounts = () => {
            localStorage.setItem('financialAccounts', JSON.stringify(accounts));
        };

        /** Salva os cartões de crédito no localStorage. */
        const saveCreditCards = () => {
            localStorage.setItem('financialCreditCards', JSON.stringify(creditCards));
        };

        /** Salva as transações no localStorage. */
        const saveTransactions = () => {
            localStorage.setItem('financialTransactions', JSON.stringify(transactions));
        };

        /** Salva as categorias no localStorage. */
        const saveCategories = () => {
            localStorage.setItem('financialCategories', JSON.stringify(categories));
        };

        // --- Gerenciamento de Contas e Cartões de Crédito ---

        // Event listener para mudança no tipo de conta (mostra/oculta limite de cheque especial)
        accountTypeSelect.addEventListener('change', () => {
            if (accountTypeSelect.value === 'bank') {
                overdraftLimitInput.classList.remove('hidden');
            } else {
                overdraftLimitInput.classList.add('hidden');
                overdraftLimitInput.value = ''; // Limpa o valor se oculto
            }
        });

        // Adiciona uma nova conta
        addAccountForm.addEventListener('submit', (e) => {
            e.preventDefault();
            const name = document.getElementById('account-name').value.trim();
            const type = document.getElementById('account-type').value;
            const initialBalance = parseFloat(initialBalanceInput.value) || 0;
            const overdraftLimit = type === 'bank' ? parseFloat(overdraftLimitInput.value) || 0 : 0;

            if (name) {
                const newAccount = {
                    id: generateUniqueId(),
                    name,
                    type,
                    balance: initialBalance,
                    overdraftLimit: overdraftLimit
                };
                accounts.push(newAccount);
                saveAccounts();
                renderAccounts();
                populateAccountSelects();
                updateSummary();
                addAccountForm.reset();
                overdraftLimitInput.classList.add('hidden');
                showFeedback('Conta adicionada com sucesso!', 'success');
            } else {
                showFeedback('Por favor, insira um nome para a conta.', 'error');
            }
        });

        // Adiciona um novo cartão de crédito
        addCreditCardForm.addEventListener('submit', (e) => {
            e.preventDefault();
            const name = document.getElementById('card-name').value.trim();
            const limit = parseFloat(document.getElementById('card-limit').value) || 0;
            const dueDay = parseInt(cardDueDayInput.value);

            if (name && limit > 0 && dueDay >= 1 && dueDay <= 31) {
                const newCard = {
                    id: generateUniqueId(),
                    name,
                    limit,
                    availableLimit: limit, // Limite disponível inicial é o limite total
                    currentBillAmount: 0, // Fatura atual começa em 0
                    dueDay // Dia do mês em que a fatura vence
                };
                creditCards.push(newCard);
                saveCreditCards();
                renderCreditCards();
                populateAccountSelects();
                addCreditCardForm.reset();
                showFeedback('Cartão de crédito adicionado com sucesso!', 'success');
            } else {
                showFeedback('Por favor, preencha todos os campos do cartão de crédito corretamente.', 'error');
            }
        });

        /** Renderiza a lista de contas na UI. */
        const renderAccounts = () => {
            accountsList.innerHTML = '';
            if (accounts.length === 0) {
                accountsList.innerHTML = '<li class="p-4 text-gray-500 text-center">Nenhuma conta adicionada ainda.</li>';
                return;
            }
            accounts.forEach(account => {
                const li = document.createElement('li');
                li.className = 'p-4 flex justify-between items-center bg-white hover:bg-gray-50';
                li.innerHTML = `
                    <div>
                        <span class="font-semibold text-gray-800">${account.name}</span>
                        <span class="text-sm text-gray-500 ml-2">(${account.type === 'bank' ? 'Banco' : 'Dinheiro'})</span>
                        <p class="text-gray-600 text-sm">Saldo: <span class="font-bold">${formatCurrency(account.balance)}</span></p>
                        ${account.type === 'bank' && account.overdraftLimit > 0 ? `<p class="text-gray-600 text-sm">Cheque Especial: ${formatCurrency(account.overdraftLimit)}</p>` : ''}
                    </div>
                    <button data-id="${account.id}" data-type="account" class="delete-btn text-red-500 hover:text-red-700 p-2 rounded-full hover:bg-red-100 transition duration-150 ease-in-out">
                        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                    </button>
                `;
                accountsList.appendChild(li);
            });
            addDeleteListeners();
        };

        /** Renderiza a lista de cartões de crédito na UI. */
        const renderCreditCards = () => {
            creditCardsList.innerHTML = '';
            if (creditCards.length === 0) {
                creditCardsList.innerHTML = '<li class="p-4 text-gray-500 text-center">Nenhum cartão de crédito adicionado ainda.</li>';
                return;
            }
            creditCards.forEach(card => {
                const li = document.createElement('li');
                li.className = 'p-4 flex justify-between items-center bg-white hover:bg-gray-50';
                li.innerHTML = `
                    <div>
                        <span class="font-semibold text-gray-800">${card.name}</span>
                        <span class="text-sm text-gray-500 ml-2">(Crédito)</span>
                        <p class="text-gray-600 text-sm">Limite: <span class="font-bold">${formatCurrency(card.limit)}</span></p>
                        <p class="text-gray-600 text-sm">Limite Disponível: <span class="font-bold">${formatCurrency(card.availableLimit)}</span></p>
                        <p class="text-gray-600 text-sm">Fatura Atual: <span class="font-bold">${formatCurrency(card.currentBillAmount)}</span></p>
                        <p class="text-gray-600 text-sm">Vencimento: Dia ${card.dueDay}</p>
                    </div>
                    <button data-id="${card.id}" data-type="creditCard" class="delete-btn text-red-500 hover:text-red-700 p-2 rounded-full hover:bg-red-100 transition duration-150 ease-in-out">
                        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                    </button>
                `;
                creditCardsList.appendChild(li);
            });
            addDeleteListeners();
        };

        /** Popula os dropdowns de seleção de conta/cartão. */
        const populateAccountSelects = () => {
            accountSelect.innerHTML = '<option value="">Selecione uma conta</option>'; // Limpa opções existentes
            // Adiciona contas bancárias/dinheiro
            accounts.forEach(account => {
                const option = document.createElement('option');
                option.value = account.id;
                option.textContent = `${account.name} (${account.type === 'bank' ? 'Conta Bancária' : 'Dinheiro'})`;
                accountSelect.appendChild(option);
            });
            // Adiciona cartões de crédito
            creditCards.forEach(card => {
                const option = document.createElement('option');
                option.value = card.id;
                option.textContent = `${card.name} (Cartão de Crédito)`;
                accountSelect.appendChild(option);
            });
        };

        /** Popula o select de categorias. */
        const populateCategorySelect = () => {
            categorySelect.innerHTML = '';
            categories.forEach(cat => {
                const option = document.createElement('option');
                option.value = cat;
                option.textContent = cat;
                categorySelect.appendChild(option);
            });
            const newOption = document.createElement('option');
            newOption.value = 'nova-categoria';
            newOption.textContent = 'Adicionar Nova Categoria...';
            categorySelect.appendChild(newOption);
        };

        // Event listener para o select de categorias (mostra/oculta input para nova categoria)
        categorySelect.addEventListener('change', () => {
            if (categorySelect.value === 'nova-categoria') {
                newCategoryInput.classList.remove('hidden');
                newCategoryInput.focus();
                newCategoryInput.value = ''; // Limpa o input anterior
            } else {
                newCategoryInput.classList.add('hidden');
                newCategoryInput.value = ''; // Limpa e oculta
            }
        });

        // --- Gerenciamento de Transações ---

        // Event listener para o envio do formulário de transação
        transactionForm.addEventListener('submit', (e) => {
            e.preventDefault();

            const date = document.getElementById('date').value;
            const description = document.getElementById('description').value.trim();
            const value = parseFloat(document.getElementById('value').value);
            const type = document.getElementById('type').value;
            const accountId = document.getElementById('account-select').value;
            const installments = parseInt(installmentsInput.value) || 1;
            const installmentsPaid = parseInt(installmentsPaidInput.value) || 0;

            let category;
            if (categorySelect.value === 'nova-categoria') {
                category = newCategoryInput.value.trim();
                if (category && !categories.includes(category)) {
                    categories.push(category); // Adiciona nova categoria à lista
                    saveCategories(); // Salva categorias atualizadas
                    populateCategorySelect(); // Repopula o select com a nova categoria
                    categorySelect.value = category; // Seleciona a categoria recém-adicionada
                } else if (!category) {
                    showFeedback('Por favor, insira um nome para a nova categoria.', 'error');
                    return;
                }
            } else {
                category = categorySelect.value;
            }

            if (!date || !description || isNaN(value) || !type || !category || !accountId) {
                showFeedback('Por favor, preencha todos os campos da transação.', 'error');
                return;
            }

            if (installmentsPaid > installments) {
                showFeedback('O número de parcelas pagas não pode ser maior que o total de parcelas.', 'error');
                return;
            }

            const account = accounts.find(acc => acc.id === accountId);
            const creditCard = creditCards.find(card => card.id === accountId);
            const isCreditCardTransaction = !!creditCard;

            // Lógica para transações parceladas
            if (installments > 1) {
                const installmentValue = value / installments;
                for (let i = 0; i < installments; i++) {
                    const transactionDate = new Date(date);
                    transactionDate.setMonth(transactionDate.getMonth() + i); // Ajusta o mês para cada parcela

                    const newTransaction = {
                        id: generateUniqueId(),
                        date: transactionDate.toISOString().split('T')[0], // Formato YYYY-MM-DD
                        description,
                        value: installmentValue,
                        type,
                        category,
                        accountId,
                        isCreditCard: isCreditCardTransaction,
                        installments: `${i + 1}/${installments}`,
                        isPaid: i < installmentsPaid // Marca como paga se o índice da parcela for menor que as parcelas já pagas
                    };
                    transactions.push(newTransaction);

                    // Atualiza saldo da conta/cartão para parcelas já pagas
                    if (newTransaction.isPaid) {
                        if (account) {
                            account.balance += (type === 'receita' ? installmentValue : -installmentValue);
                        } else if (creditCard) {
                            // Para cartão de crédito, transações pagas (estornos) afetam a fatura e limite disponível
                            creditCard.currentBillAmount -= (type === 'receita' ? installmentValue : -installmentValue);
                            creditCard.availableLimit += (type === 'receita' ? installmentValue : -installmentValue);
                        }
                    } else if (isCreditCardTransaction && type === 'despesa') {
                        // Se for despesa de cartão e não paga, afeta a fatura e limite disponível
                        creditCard.currentBillAmount += installmentValue;
                        creditCard.availableLimit -= installmentValue;
                    } else if (isCreditCardTransaction && type === 'receita' && !newTransaction.isPaid) {
                        // Se for receita de cartão e não paga (estorno futuro), afeta a fatura e limite disponível
                        creditCard.currentBillAmount -= installmentValue;
                        creditCard.availableLimit += installmentValue;
                    }
                }
            } else { // Lógica para transações de parcela única
                const newTransaction = {
                    id: generateUniqueId(),
                    date,
                    description,
                    value,
                    type,
                    category,
                    accountId,
                    isCreditCard: isCreditCardTransaction,
                    installments: '1/1',
                    isPaid: installmentsPaid > 0 // Marca como paga se 1 parcela e installmentsPaid > 0
                };
                transactions.push(newTransaction);

                // Atualiza saldo da conta/cartão para transação única se paga
                if (newTransaction.isPaid) {
                    if (account) {
                        account.balance += (type === 'receita' ? value : -value);
                    } else if (creditCard) {
                        // Para cartão de crédito, transações pagas (estornos) afetam a fatura e limite disponível
                        creditCard.currentBillAmount -= (type === 'receita' ? value : -value);
                        creditCard.availableLimit += (type === 'receita' ? value : -value);
                    }
                } else if (isCreditCardTransaction && type === 'despesa') {
                    // Se for despesa de cartão e não paga, afeta a fatura e limite disponível
                    creditCard.currentBillAmount += value;
                    creditCard.availableLimit -= value;
                } else if (isCreditCardTransaction && type === 'receita' && !newTransaction.isPaid) {
                    // Se for receita de cartão e não paga (estorno futuro), afeta a fatura e limite disponível
                    creditCard.currentBillAmount -= value;
                    creditCard.availableLimit += value;
                }
            }

            // Salva os dados atualizados e renderiza a UI
            saveTransactions();
            saveAccounts();
            saveCreditCards();
            renderTransactions();
            updateSummary();
            calculateNextMonthExpensesPrediction();
            renderMonthlySummary();
            renderExpensesChart();
            renderDetailedCardBills();
            checkCreditCardDueDates(); // Verifica datas de vencimento após nova transação
            transactionForm.reset();
            installmentsInput.value = 1; // Reseta parcelas para 1
            installmentsPaidInput.value = 0; // Reseta parcelas pagas para 0
            newCategoryInput.classList.add('hidden'); // Oculta input de nova categoria
            populateCategorySelect(); // Repopula o select para resetá-lo
            showFeedback('Transação adicionada com sucesso!', 'success');
        });

        /** Renderiza a tabela de transações na UI para o mês e ano atuais. */
        const renderTransactions = () => {
            transactionList.innerHTML = '';
            // Filtra e ordena transações pelo mês e ano atuais
            const filteredTransactions = transactions.filter(t => {
                const transactionDate = new Date(t.date);
                return transactionDate.getMonth() === currentMonth && transactionDate.getFullYear() === currentYear;
            }).sort((a, b) => new Date(a.date) - new Date(b.date)); // Ordena por data

            if (filteredTransactions.length === 0) {
                noTransactionsMessage.classList.remove('hidden');
                return;
            } else {
                noTransactionsMessage.classList.add('hidden');
            }

            filteredTransactions.forEach(transaction => {
                const tr = document.createElement('tr');
                // Aplica estilo de "pago" se a transação estiver marcada como tal
                tr.className = `bg-white hover:bg-gray-50 ${transaction.isPaid ? 'paid-transaction' : ''}`;

                // Obtém o nome da conta ou cartão associado à transação
                const accountOrCardName = transaction.isCreditCard
                    ? creditCards.find(card => card.id === transaction.accountId)?.name || 'Cartão Desconhecido'
                    : accounts.find(acc => acc.id === transaction.accountId)?.name || 'Conta Desconhecida';

                tr.innerHTML = `
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">${new Date(transaction.date).toLocaleDateString('pt-BR')}</td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">${transaction.description}</td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm font-medium ${transaction.type === 'receita' ? 'text-green-600' : 'text-red-600'}">
                        ${formatCurrency(transaction.value)}
                    </td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">${transaction.type === 'receita' ? 'Receita' : 'Despesa'}</td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">${transaction.category}</td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">${accountOrCardName}</td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">${transaction.installments}</td>
                    <td class="px-6 py-4 whitespace-nowrap text-right text-sm font-medium">
                        ${!transaction.isPaid ? `<button data-id="${transaction.id}" data-type="transaction" class="pay-transaction-btn text-blue-600 hover:text-blue-900 mr-2 p-2 rounded-full hover:bg-blue-100 transition duration-150 ease-in-out" title="Marcar como Pago">
                            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg>
                        </button>` : ''}
                        <button data-id="${transaction.id}" data-type="transaction" class="delete-btn text-red-500 hover:text-red-700 p-2 rounded-full hover:bg-red-100 transition duration-150 ease-in-out" title="Excluir Transação">
                            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                        </button>
                    </td>
                `;
                transactionList.appendChild(tr);
            });
            addDeleteListeners();
            addPayTransactionListeners(); // Adiciona listeners para o botão "Pagar"
        };

        /**
         * Executa a exclusão de uma conta, cartão ou transação.
         * @param {string} id - O ID do item a ser excluído.
         * @param {'account'|'creditCard'|'transaction'} type - O tipo do item.
         */
        const performDelete = (id, type) => {
            if (type === 'account') {
                // Verifica se a conta possui transações associadas
                const hasTransactions = transactions.some(t => t.accountId === id);
                if (hasTransactions) {
                    showFeedback('Não é possível excluir uma conta que possui transações associadas. Exclua as transações primeiro.', 'error');
                } else {
                    accounts = accounts.filter(acc => acc.id !== id);
                    saveAccounts();
                    renderAccounts();
                    populateAccountSelects();
                    updateSummary();
                    showFeedback('Conta excluída com sucesso!', 'success');
                }
            } else if (type === 'creditCard') {
                // Verifica se o cartão de crédito possui transações associadas
                const hasTransactions = transactions.some(t => t.accountId === id);
                if (hasTransactions) {
                    showFeedback('Não é possível excluir um cartão de crédito que possui transações associadas. Exclua as transações primeiro.', 'error');
                } else {
                    creditCards = creditCards.filter(card => card.id !== id);
                    saveCreditCards();
                    renderCreditCards();
                    populateAccountSelects();
                    updateSummary();
                    showFeedback('Cartão de crédito excluído com sucesso!', 'success');
                }
            } else if (type === 'transaction') {
                const deletedTransaction = transactions.find(t => t.id === id);
                if (deletedTransaction) {
                    // Reverte o impacto da transação no saldo da conta/cartão ao ser excluída
                    if (deletedTransaction.isCreditCard) {
                        const card = creditCards.find(c => c.id === deletedTransaction.accountId);
                        if (card) {
                            // Se a transação era uma despesa, subtrai da fatura e adiciona ao limite disponível
                            // Se era uma receita, adiciona à fatura e subtrai do limite disponível
                            card.currentBillAmount -= (deletedTransaction.type === 'receita' ? -deletedTransaction.value : deletedTransaction.value);
                            card.availableLimit += (deletedTransaction.type === 'receita' ? -deletedTransaction.value : deletedTransaction.value);
                            saveCreditCards();
                        }
                    } else {
                        // Para contas bancárias/dinheiro, reverte o saldo se a transação estava marcada como paga
                        if (deletedTransaction.isPaid) {
                            const account = accounts.find(acc => acc.id === deletedTransaction.accountId);
                            if (account) {
                                account.balance -= (deletedTransaction.type === 'receita' ? deletedTransaction.value : -deletedTransaction.value);
                                saveAccounts();
                            }
                        }
                    }
                    transactions = transactions.filter(t => t.id !== id);
                    saveTransactions();
                    // Re-renderiza e atualiza a UI após a exclusão
                    renderTransactions();
                    updateSummary();
                    calculateNextMonthExpensesPrediction();
                    renderMonthlySummary();
                    renderExpensesChart();
                    renderDetailedCardBills();
                    checkCreditCardDueDates();
                    showFeedback('Transação excluída com sucesso!', 'success');
                }
            }
        };

        /** Adiciona event listeners para os botões de exclusão. */
        const addDeleteListeners = () => {
            document.querySelectorAll('.delete-btn').forEach(button => {
                button.onclick = (e) => {
                    const id = e.currentTarget.dataset.id;
                    const type = e.currentTarget.dataset.type;
                    performDelete(id, type); // Chama a função de exclusão diretamente
                };
            });
        };

        /** Adiciona event listeners para os botões de "Marcar como Pago". */
        const addPayTransactionListeners = () => {
            document.querySelectorAll('.pay-transaction-btn').forEach(button => {
                button.onclick = (e) => {
                    const id = e.currentTarget.dataset.id;
                    markTransactionAsPaid(id);
                };
            });
        };

        /**
         * Marca uma transação como paga e atualiza os saldos.
         * @param {string} transactionId - O ID da transação a ser marcada.
         */
        const markTransactionAsPaid = (transactionId) => {
            const transactionIndex = transactions.findIndex(t => t.id === transactionId);
            if (transactionIndex > -1) {
                const transaction = transactions[transactionIndex];
                if (!transaction.isPaid) {
                    transaction.isPaid = true;

                    // Para transações de conta/dinheiro, atualiza o saldo da conta
                    // Para transações de cartão de crédito, o 'pago' significa que a transação foi processada
                    // e incluída na fatura. O saldo da fatura e limite disponível são gerenciados
                    // na adição/exclusão da transação, não neste ponto.
                    if (!transaction.isCreditCard) {
                        const account = accounts.find(acc => acc.id === transaction.accountId);
                        if (account) {
                            account.balance += (transaction.type === 'receita' ? transaction.value : -transaction.value);
                            saveAccounts();
                        }
                    }

                    saveTransactions();
                    // Re-renderiza e atualiza a UI para refletir o status de pago
                    renderTransactions();
                    updateSummary();
                    renderMonthlySummary();
                    renderExpensesChart();
                    renderDetailedCardBills();
                    checkCreditCardDueDates();
                    showFeedback('Transação marcada como paga e saldo atualizado (se aplicável)!', 'success');
                } else {
                    showFeedback('Esta transação já está marcada como paga.', 'info');
                }
            }
        };

        // --- Resumo e Previsão ---

        /** Atualiza os valores do resumo principal (Total de Receitas, Despesas, Saldo Atual). */
        const updateSummary = () => {
            let totalReceitasPoderDeCompra = 0; // Total de fundos disponíveis / poder de compra

            // Soma saldos positivos das contas e limites de cheque especial
            accounts.forEach(account => {
                totalReceitasPoderDeCompra += account.balance > 0 ? account.balance : 0;
                totalReceitasPoderDeCompra += account.overdraftLimit;
            });

            // Soma limites disponíveis dos cartões de crédito
            creditCards.forEach(card => {
                totalReceitasPoderDeCompra += card.availableLimit;
            });

            // Obtém o total de despesas para o mês atual (calculado por renderMonthlySummary)
            // Este será o valor exibido em "Total de Despesas (Mês Atual)"
            const totalDespesasMesAtual = parseFloat(monthlyExpensesEl.textContent.replace('R$', '').replace('.', '').replace(',', '.')) || 0;

            // Calcula o saldo atual (poder de compra - despesas do mês atual)
            const saldoAtual = totalReceitasPoderDeCompra - totalDespesasMesAtual;

            totalReceitasEl.textContent = formatCurrency(totalReceitasPoderDeCompra);
            totalDespesasEl.textContent = formatCurrency(totalDespesasMesAtual);
            saldoAtualEl.textContent = formatCurrency(saldoAtual);

            // Calcula o total das faturas de cartão de crédito para o mês atual
            let totalFaturasCartaoMesAtual = 0;
            creditCards.forEach(card => {
                totalFaturasCartaoMesAtual += card.currentBillAmount;
            });
            totalCardBillsEl.textContent = formatCurrency(totalFaturasCartaoMesAtual);

            // Altera a cor do saldoAtual com base no valor
            if (saldoAtual >= 0) {
                saldoAtualEl.classList.remove('text-red-700');
                saldoAtualEl.classList.add('text-green-700');
            } else {
                saldoAtualEl.classList.remove('text-green-700');
                saldoAtualEl.classList.add('text-red-700');
            }
        };

        /** Calcula a previsão de despesas para o próximo mês. */
        const calculateNextMonthExpensesPrediction = () => {
            let predictedExpenses = 0;
            const nextMonth = (currentMonth + 1) % 12;
            const nextYear = currentMonth === 11 ? currentYear + 1 : currentYear;

            // Inclui as faturas de cartão de crédito como despesas previstas
            creditCards.forEach(card => {
                predictedExpenses += card.currentBillAmount;
            });

            transactions.forEach(t => {
                // Considera apenas despesas que NÃO estão pagas e caem no próximo mês
                // E que não são de cartão de crédito (já incluídas acima na fatura)
                if (t.type === 'despesa' && !t.isPaid) {
                    const transactionDate = new Date(t.date);
                    // Verifica se a transação é para o próximo mês
                    if (transactionDate.getMonth() === nextMonth && transactionDate.getFullYear() === nextYear) {
                        // Se for uma despesa de cartão, já foi incluída na fatura. Evita duplicidade.
                        if (!t.isCreditCard) {
                            predictedExpenses += t.value;
                        }
                    } else if (t.installments && t.installments !== '1/1') {
                        // Para transações parceladas, verifica se alguma parcela futura cai no próximo mês
                        const [currentInstallmentNum, totalInstallmentsNum] = t.installments.split('/').map(Number);
                        const originalDate = new Date(t.date);

                        // Itera pelas parcelas futuras a partir da próxima parcela não paga
                        for (let i = currentInstallmentNum; i <= totalInstallmentsNum; i++) {
                            const installmentDate = new Date(originalDate);
                            installmentDate.setMonth(originalDate.getMonth() + (i - 1)); // Ajusta o mês para cada parcela

                            if (installmentDate.getMonth() === nextMonth && installmentDate.getFullYear() === nextYear) {
                                // Se for uma despesa de cartão, já foi incluída na fatura. Evita duplicidade.
                                if (!t.isCreditCard) {
                                    predictedExpenses += t.value; // Adiciona o valor da parcela
                                }
                                break; // Adiciona apenas uma vez por transação por mês
                            }
                        }
                    }
                }
            });
            predictedExpensesEl.textContent = formatCurrency(predictedExpenses);
        };


        // --- Navegação do Resumo Mensal ---

        /** Atualiza a exibição do mês e ano atuais no resumo mensal. */
        const updateMonthlyDisplay = () => {
            const date = new Date(currentYear, currentMonth);
            currentMonthDisplay.textContent = date.toLocaleDateString('pt-BR', { month: 'long', year: 'numeric' });
        };

        /** Renderiza o resumo mensal (Receitas, Despesas, Saldo do Mês). */
        const renderMonthlySummary = () => {
            let monthlyIncome = 0;
            let monthlyExpenses = 0;

            // Calcula o total de fundos disponíveis para o mês atual
            // Inclui saldos de contas, limites de cheque especial e limites disponíveis de cartões.
            let monthlyAvailableFunds = 0;
            accounts.forEach(account => {
                monthlyAvailableFunds += account.balance > 0 ? account.balance : 0;
                monthlyAvailableFunds += account.overdraftLimit;
            });
            creditCards.forEach(card => {
                monthlyAvailableFunds += card.availableLimit;
            });
            // Adiciona transações de receita reais para o mês atual
            transactions.forEach(t => {
                const transactionDate = new Date(t.date);
                if (t.type === 'receita' && transactionDate.getMonth() === currentMonth && transactionDate.getFullYear() === currentYear) {
                    monthlyAvailableFunds += t.value;
                }
            });

            monthlyIncome = monthlyAvailableFunds; // Atribui os fundos disponíveis à "Receitas do Mês"

            transactions.forEach(t => {
                const transactionDate = new Date(t.date);
                if (transactionDate.getMonth() === currentMonth && transactionDate.getFullYear() === currentYear) {
                    if (t.type === 'despesa') {
                        // Soma TODAS as despesas para o mês atual, independentemente do status de pago
                        monthlyExpenses += t.value;
                    }
                }
            });

            monthlyIncomeEl.textContent = formatCurrency(monthlyIncome);
            monthlyExpensesEl.textContent = formatCurrency(monthlyExpenses);
            monthlyBalanceEl.textContent = formatCurrency(monthlyIncome - monthlyExpenses);

            // Atualiza o gráfico de despesas
            renderExpensesChart();
        };

        // Botão para navegar para o mês anterior
        prevMonthBtn.addEventListener('click', () => {
            currentMonth--;
            if (currentMonth < 0) {
                currentMonth = 11;
                currentYear--;
            }
            updateMonthlyDisplay();
            renderTransactions();
            renderMonthlySummary();
            updateSummary(); // Recalcula o resumo principal ao mudar o mês
            calculateNextMonthExpensesPrediction(); // Recalcula a previsão
        });

        // Botão para navegar para o próximo mês
        nextMonthBtn.addEventListener('click', () => {
            currentMonth++;
            if (currentMonth > 11) {
                currentMonth = 0;
                currentYear++;
            }
            updateMonthlyDisplay();
            renderTransactions();
            renderMonthlySummary();
            updateSummary(); // Recalcula o resumo principal ao mudar o mês
            calculateNextMonthExpensesPrediction(); // Recalcula a previsão
        });

        // --- Funcionalidade de Importação ---

        // Alterna a visibilidade da seção de importação
        importHeader.addEventListener('click', () => {
            importSectionContent.classList.toggle('hidden');
            importArrow.classList.toggle('rotated');
        });

        // Event listener para o botão de importar arquivo
        importFileBtn.addEventListener('click', async () => {
            const file = fileInput.files[0];
            if (!file) {
                showFeedback('Por favor, selecione um arquivo para importar.', 'error');
                return;
            }

            importedTransactionsToReview = []; // Limpa dados de revisão anteriores
            importedTransactionsTableBody.innerHTML = ''; // Limpa a tabela

            const reader = new FileReader();
            reader.onload = async (e) => {
                const data = e.target.result;
                const fileExtension = file.name.split('.').pop().toLowerCase();

                try {
                    if (fileExtension === 'ofx') {
                        importedTransactionsToReview = parseOFX(data);
                    } else if (['csv', 'xls', 'xlsx'].includes(fileExtension)) {
                        importedTransactionsToReview = parseExcelCsv(data, fileExtension);
                    } else {
                        showFeedback('Formato de arquivo não suportado. Por favor, use OFX, CSV ou Excel.', 'error');
                        return;
                    }

                    if (importedTransactionsToReview.length > 0) {
                        renderImportedTransactionsForReview();
                        importReviewSection.classList.remove('hidden');
                        showFeedback('Arquivo carregado com sucesso! Revise as transações.', 'success');
                    } else {
                        showFeedback('Nenhuma transação válida encontrada no arquivo.', 'info');
                        importReviewSection.classList.add('hidden');
                    }
                } catch (error) {
                    console.error('Erro ao processar o arquivo:', error);
                    showFeedback('Erro ao processar o arquivo. Verifique o formato.', 'error');
                    importReviewSection.classList.add('hidden');
                }
            };

            if (fileExtension === 'ofx') {
                reader.readAsText(file); // OFX é baseado em texto (XML)
            } else {
                reader.readAsBinaryString(file); // Excel/CSV precisam de string binária
            }
        });

        /**
         * Analisa o conteúdo de um arquivo OFX (simplificado).
         * @param {string} ofxContent - O conteúdo do arquivo OFX.
         * @returns {Array<Object>} Um array de objetos de transação.
         */
        const parseOFX = (ofxContent) => {
            const parsedTransactions = [];
            const transactionRegex = /<STMTTRN>([\s\S]*?)<\/STMTTRN>/g;
            let match;

            while ((match = transactionRegex.exec(ofxContent)) !== null) {
                const transactionBlock = match[1];
                const dateMatch = /<DTPOSTED>(\d{8})/.exec(transactionBlock);
                const descriptionMatch = /<MEMO>([\s\S]*?)<\/(MEMO|NAME)>/.exec(transactionBlock);
                const valueMatch = /<TRNAMT>([\d.-]+)/.exec(transactionBlock);
                const typeMatch = /<TRNTYPE>(DEBIT|CREDIT)/.exec(transactionBlock);

                if (dateMatch && descriptionMatch && valueMatch && typeMatch) {
                    const date = dateMatch[1];
                    const year = date.substring(0, 4);
                    const month = date.substring(4, 6);
                    const day = date.substring(6, 8);
                    const formattedDate = `${year}-${month}-${day}`;

                    const description = descriptionMatch[1].trim();
                    const value = parseFloat(valueMatch[1]);
                    const type = typeMatch[1] === 'CREDIT' ? 'receita' : 'despesa';

                    parsedTransactions.push({
                        id: generateUniqueId(),
                        date: formattedDate,
                        description,
                        value: Math.abs(value), // Garante valor positivo
                        type,
                        category: 'Importado', // Categoria padrão para importados
                        accountId: '', // Será definido durante a revisão
                        isCreditCard: false, // Padrão, usuário pode mudar se necessário
                        installments: '1/1',
                        isPaid: true // Transações importadas geralmente já estão pagas/processadas
                    });
                }
            }
            return parsedTransactions;
        };

        /**
         * Analisa o conteúdo de um arquivo Excel/CSV usando SheetJS.
         * @param {string} data - O conteúdo binário do arquivo.
         * @param {string} fileExtension - A extensão do arquivo ('csv', 'xls', 'xlsx').
         * @returns {Array<Object>} Um array de objetos de transação.
         */
        const parseExcelCsv = (data, fileExtension) => {
            const workbook = XLSX.read(data, { type: 'binary' });
            const sheetName = workbook.SheetNames[0];
            const worksheet = workbook.Sheets[sheetName];
            const json = XLSX.utils.sheet_to_json(worksheet, { header: 1 });

            const parsedTransactions = [];
            if (json.length === 0) return parsedTransactions;

            // Tenta encontrar a linha do cabeçalho
            let headerRowIndex = -1;
            let dateCol = -1, descCol = -1, valueCol = -1, typeCol = -1;

            for (let i = 0; i < json.length; i++) {
                const row = json[i];
                if (Array.isArray(row)) {
                    const lowerCaseRow = row.map(cell => typeof cell === 'string' ? cell.toLowerCase().trim() : '');
                    dateCol = lowerCaseRow.indexOf('data');
                    if (dateCol === -1) dateCol = lowerCaseRow.indexOf('date');
                    
                    descCol = lowerCaseRow.indexOf('descricao');
                    if (descCol === -1) descCol = lowerCaseRow.indexOf('description');

                    valueCol = lowerCaseRow.indexOf('valor');
                    if (valueCol === -1) valueCol = lowerCaseRow.indexOf('value');

                    typeCol = lowerCaseRow.indexOf('tipo');
                    if (typeCol === -1) typeCol = lowerCaseRow.indexOf('type');

                    if (dateCol !== -1 && descCol !== -1 && valueCol !== -1 && typeCol !== -1) {
                        headerRowIndex = i;
                        break;
                    }
                }
            }

            if (headerRowIndex === -1) {
                showFeedback('Não foi possível identificar as colunas "Data", "Descrição", "Valor" e "Tipo" no seu arquivo.', 'error');
                return parsedTransactions;
            }

            for (let i = headerRowIndex + 1; i < json.length; i++) {
                const row = json[i];
                if (!Array.isArray(row)) continue;

                const dateRaw = row[dateCol];
                const description = row[descCol];
                const valueRaw = row[valueCol];
                const typeRaw = row[typeCol];

                if (dateRaw && description && valueRaw !== undefined && typeRaw) {
                    let date;
                    // Lida com diferentes formatos de data (número Excel, string)
                    if (typeof dateRaw === 'number') {
                        date = new Date(Math.round((dateRaw - 25569) * 86400 * 1000));
                    } else {
                        date = new Date(dateRaw);
                    }
                    
                    const formattedDate = date.toISOString().split('T')[0]; // YYYY-MM-DD

                    const value = parseFloat(String(valueRaw).replace(',', '.')); // Lida com vírgula como separador decimal
                    const type = String(typeRaw).toLowerCase().includes('receita') || String(typeRaw).toLowerCase().includes('credit') ? 'receita' : 'despesa';

                    if (!isNaN(value) && formattedDate !== 'Invalid Date') {
                        parsedTransactions.push({
                            id: generateUniqueId(),
                            date: formattedDate,
                            description: String(description).trim(),
                            value: Math.abs(value),
                            type,
                            category: 'Importado',
                            accountId: '', // Será selecionado pelo usuário
                            isCreditCard: false,
                            installments: '1/1',
                            isPaid: true
                        });
                    }
                }
            }
            return parsedTransactions;
        };


        /** Renderiza as transações importadas na tabela de revisão. */
        const renderImportedTransactionsForReview = () => {
            importedTransactionsTableBody.innerHTML = '';
            importedTransactionsToReview.forEach((transaction, index) => {
                const tr = document.createElement('tr');
                tr.className = 'bg-white hover:bg-gray-50';

                // Cria opções para os selects de conta/cartão
                const accountSelectOptions = accounts.map(acc => `<option value="${acc.id}">${acc.name} (${acc.type === 'bank' ? 'Conta Bancária' : 'Dinheiro'})</option>`).join('');
                const creditCardSelectOptions = creditCards.map(card => `<option value="${card.id}">${card.name} (Cartão de Crédito)</option>`).join('');

                tr.innerHTML = `
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">${new Date(transaction.date).toLocaleDateString('pt-BR')}</td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">${transaction.description}</td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm font-medium ${transaction.type === 'receita' ? 'text-green-600' : 'text-red-600'}">
                        ${formatCurrency(transaction.value)}
                    </td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">${transaction.type === 'receita' ? 'Receita' : 'Despesa'}</td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
                        <input type="text" class="category-input mt-1 block w-full px-2 py-1 border border-gray-300 rounded-md shadow-sm text-sm" value="${transaction.category}" data-index="${index}">
                    </td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
                        <select class="account-select-review mt-1 block w-full px-2 py-1 border border-gray-300 rounded-md shadow-sm text-sm" data-index="${index}">
                            <option value="">Selecione</option>
                            ${accountSelectOptions}
                            ${creditCardSelectOptions}
                        </select>
                    </td>
                `;
                importedTransactionsTableBody.appendChild(tr);
            });

            // Adiciona event listeners para os inputs de categoria e selects de conta na tabela de revisão
            document.querySelectorAll('.category-input').forEach(input => {
                input.addEventListener('change', (e) => {
                    const index = parseInt(e.target.dataset.index);
                    importedTransactionsToReview[index].category = e.target.value.trim();
                });
            });

            document.querySelectorAll('.account-select-review').forEach(select => {
                select.addEventListener('change', (e) => {
                    const index = parseInt(e.target.dataset.index);
                    const selectedAccountId = e.target.value;
                    importedTransactionsToReview[index].accountId = selectedAccountId;
                    // Determina se é uma transação de cartão de crédito com base na conta selecionada
                    importedTransactionsToReview[index].isCreditCard = creditCards.some(card => card.id === selectedAccountId);
                });
            });
        };

        // Confirma a importação das transações revisadas
        confirmImportBtn.addEventListener('click', () => {
            let allAccountsAssigned = true;
            importedTransactionsToReview.forEach(t => {
                if (!t.accountId) {
                    allAccountsAssigned = false;
                }
            });

            if (!allAccountsAssigned) {
                showFeedback('Por favor, vincule todas as transações a uma conta ou cartão.', 'error');
                return;
            }

            importedTransactionsToReview.forEach(newTransaction => {
                transactions.push(newTransaction);

                // Atualiza o saldo da conta/cartão para transações importadas (assumidas como pagas)
                const account = accounts.find(acc => acc.id === newTransaction.accountId);
                const creditCard = creditCards.find(card => card.id === newTransaction.accountId);

                if (account) {
                    account.balance += (newTransaction.type === 'receita' ? newTransaction.value : -newTransaction.value);
                } else if (creditCard) {
                    // Para transações de cartão de crédito importadas, elas já deveriam ter afetado
                    // a fatura e o limite disponível no extrato original.
                    // Aqui, garantimos que os valores internos do cartão reflitam essa transação.
                    creditCard.currentBillAmount += (newTransaction.type === 'despesa' ? newTransaction.value : -newTransaction.value);
                    creditCard.availableLimit -= (newTransaction.type === 'despesa' ? newTransaction.value : -newTransaction.value);
                }
            });

            saveTransactions();
            saveAccounts();
            saveCreditCards();
            // Re-renderiza e atualiza a UI após a importação
            renderTransactions();
            updateSummary();
            calculateNextMonthExpensesPrediction();
            renderMonthlySummary();
            renderExpensesChart();
            renderDetailedCardBills();
            checkCreditCardDueDates();
            importedTransactionsToReview = []; // Limpa o array de revisão
            importedTransactionsTableBody.innerHTML = ''; // Limpa a tabela
            importReviewSection.classList.add('hidden'); // Oculta a seção de revisão
            fileInput.value = ''; // Limpa o input de arquivo
            showFeedback('Transações importadas e adicionadas com sucesso!', 'success');
        });

        // Cancela a revisão da importação
        cancelReviewBtn.addEventListener('click', () => {
            importedTransactionsToReview = [];
            importedTransactionsTableBody.innerHTML = '';
            importReviewSection.classList.add('hidden');
            fileInput.value = '';
            showFeedback('Importação cancelada.', 'info');
        });


        // --- Integração Chart.js (Gráfico de Despesas por Categoria) ---

        /** Renderiza/atualiza o gráfico de despesas por categoria. */
        const renderExpensesChart = () => {
            const ctx = document.getElementById('expenses-chart').getContext('2d');

            // Filtra transações para o mês e ano atuais, incluindo TODAS as despesas
            const monthlyExpensesForChart = transactions.filter(t => {
                const transactionDate = new Date(t.date);
                return t.type === 'despesa' &&
                       transactionDate.getMonth() === currentMonth &&
                       transactionDate.getFullYear() === currentYear;
            });

            // Agrega despesas por categoria
            const expensesByCategory = monthlyExpensesForChart.reduce((acc, transaction) => {
                const category = transaction.category || 'Outros';
                acc[category] = (acc[category] || 0) + transaction.value;
                return acc;
            }, {});

            const labels = Object.keys(expensesByCategory);
            const data = Object.values(expensesByCategory);

            // Cores discretas predefinidas para o gráfico
            const discreetColors = [
                '#AEC6CF', // Azul Acinzentado Claro
                '#B3E0FF', // Azul Muito Claro
                '#ADD8E6', // Azul Claro
                '#87CEEB', // Azul Céu
                '#6495ED', // Azul Centáurea
                '#4682B4', // Azul Aço
                '#5F9EA0', // Azul Cadete
                '#778899', // Cinza Ardósia Claro
                '#B0C4DE', // Azul Aço Claro
                '#C6E2FF'  // Azul Mais Claro
            ];

            // Atribui cores ciclicamente
            const backgroundColors = labels.map((_, index) => discreetColors[index % discreetColors.length]);
            // Escurece ligeiramente as cores de fundo para as bordas
            const borderColors = backgroundColors.map(color => {
                const hex = color.replace('#', '');
                const r = parseInt(hex.substring(0, 2), 16);
                const g = parseInt(hex.substring(2, 4), 16);
                const b = parseInt(hex.substring(4, 6), 16);
                return `rgb(${r * 0.8}, ${g * 0.8}, ${b * 0.8})`;
            });

            if (expensesChart) {
                expensesChart.data.labels = labels;
                expensesChart.data.datasets[0].data = data;
                expensesChart.data.datasets[0].backgroundColor = backgroundColors;
                expensesChart.data.datasets[0].borderColor = borderColors;
                expensesChart.update();
            } else {
                expensesChart = new Chart(ctx, {
                    type: 'pie', // Tipo de gráfico: pizza
                    data: {
                        labels: labels,
                        datasets: [{
                            label: 'Despesas por Categoria',
                            data: data,
                            backgroundColor: backgroundColors,
                            borderColor: borderColors,
                            borderWidth: 1
                        }]
                    },
                    options: {
                        responsive: true,
                        plugins: {
                            legend: {
                                position: 'top',
                            },
                            title: {
                                display: true,
                                text: 'Despesas por Categoria no Mês Atual' // Título do gráfico
                            }
                        }
                    }
                });
            }
        };

        // --- Funcionalidade de Download ---

        // Event listener para o botão de baixar planilha
        downloadXlsxBtn.addEventListener('click', () => {
            // Filtra transações do mês e ano atuais para exportação
            const dataToExport = transactions.filter(t => {
                const transactionDate = new Date(t.date);
                return transactionDate.getMonth() === currentMonth && transactionDate.getFullYear() === currentYear;
            }).map(t => ({
                Data: new Date(t.date).toLocaleDateString('pt-BR'),
                Descrição: t.description,
                Valor: t.value,
                Tipo: t.type === 'receita' ? 'Receita' : 'Despesa',
                Categoria: t.category,
                'Conta / Cartão': t.isCreditCard
                    ? creditCards.find(card => card.id === t.accountId)?.name || 'Cartão Desconhecido'
                    : accounts.find(acc => acc.id === t.accountId)?.name || 'Conta Desconhecida',
                Parcela: t.installments,
                Pago: t.isPaid ? 'Sim' : 'Não'
            }));

            if (dataToExport.length === 0) {
                showFeedback('Não há transações para baixar no mês atual.', 'info');
                return;
            }

            // Cria a planilha e o arquivo XLSX
            const worksheet = XLSX.utils.json_to_sheet(dataToExport);
            const workbook = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(workbook, worksheet, "Transacoes");

            const monthName = new Date(currentYear, currentMonth).toLocaleDateString('pt-BR', { month: 'long' });
            const fileName = `transacoes_${monthName}_${currentYear}.xlsx`;
            XLSX.writeFile(workbook, fileName);

            showFeedback('Planilha baixada com sucesso!', 'success');
        });

        // --- Faturas Detalhadas de Cartão ---
        // Alterna a visibilidade da seção de faturas detalhadas
        detailedCardBillsHeader.addEventListener('click', () => {
            detailedCardBillsContent.classList.toggle('hidden');
            detailedCardBillsArrow.classList.toggle('rotated');
        });

        /** Renderiza a lista de faturas detalhadas de cartão. */
        const renderDetailedCardBills = () => {
            detailedCardBillsList.innerHTML = '';
            if (creditCards.length === 0) {
                noCardBillsMessage.classList.remove('hidden');
                return;
            } else {
                noCardBillsMessage.classList.add('hidden');
            }

            creditCards.forEach(card => {
                const li = document.createElement('li');
                li.className = 'p-4 flex justify-between items-center bg-white hover:bg-gray-50';
                li.innerHTML = `
                    <div class="font-semibold text-gray-800">${card.name}: <span class="text-purple-700">${formatCurrency(card.currentBillAmount)}</span></div>
                    <div class="text-sm text-gray-600">Vencimento: Dia ${card.dueDay}</div>
                `;
                detailedCardBillsList.appendChild(li);
            });
        };

        // --- Função para verificar datas de vencimento de cartão e fechamento de fatura ---
        /**
         * Verifica as datas de vencimento e fechamento de fatura dos cartões de crédito
         * e dispara notificações se as condições forem atendidas.
         */
        const checkCreditCardDueDates = () => {
            const today = new Date();
            today.setHours(0, 0, 0, 0); // Normaliza para o início do dia para comparação de datas

            creditCards.forEach(card => {
                const dueDay = card.dueDay;

                // Calcula a data de vencimento para o ciclo atual da fatura
                let billDueDate = new Date(today.getFullYear(), today.getMonth(), dueDay);
                billDueDate.setHours(0, 0, 0, 0);

                // Se o dia atual já passou o dia de vencimento para este mês,
                // a data de vencimento da fatura atual deve ser para o próximo ciclo.
                if (today.getDate() > dueDay) {
                    billDueDate.setMonth(today.getMonth() + 1);
                }

                // Calcula a data de fechamento da fatura (10 dias antes do vencimento)
                let billClosingDate = new Date(billDueDate.getTime());
                billClosingDate.setDate(billDueDate.getDate() - 10);
                billClosingDate.setHours(0, 0, 0, 0);

                // Verifica e dispara notificação de fechamento de fatura
                // A notificação ocorre APENAS no dia exato do fechamento.
                if (today.getTime() === billClosingDate.getTime()) {
                    showFeedback(`A fatura do cartão ${card.name} fechou hoje! Valor: ${formatCurrency(card.currentBillAmount)}. Vencimento: ${billDueDate.toLocaleDateString('pt-BR')}`, 'info');
                }

                // Verifica e dispara notificação de vencimento (lógica existente)
                // Calcula a diferença em dias entre hoje e a próxima data de vencimento
                const diffTimeDue = billDueDate.getTime() - today.getTime();
                const diffDaysDue = Math.ceil(diffTimeDue / (1000 * 60 * 60 * 24));

                // Dispara notificação se a data de vencimento está dentro dos próximos 7 dias (inclusive hoje)
                // e a data de vencimento ainda não passou.
                if (diffDaysDue <= 7 && diffDaysDue >= 0) {
                    if (diffDaysDue === 0) {
                        showFeedback(`A fatura do cartão ${card.name} vence hoje! Valor: ${formatCurrency(card.currentBillAmount)}`, 'info');
                    } else {
                        showFeedback(`A fatura do cartão ${card.name} vence em ${diffDaysDue} dia(s)! Valor: ${formatCurrency(card.currentBillAmount)}`, 'info');
                    }
                }
            });
        };


        // Carregamento inicial dos dados quando a página é carregada
        window.onload = () => {
            loadData();
            updateMonthlyDisplay(); // Define a exibição inicial do mês
        };
    </script>
</body>
</html>
