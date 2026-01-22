# z7qmj9qg28-rgb.github.io
[App.tsx](https://github.com/user-attachments/files/24784350/App.tsx)

import React, { useState, useEffect, useMemo } from 'react';
import Dashboard from './components/Dashboard';
import TransactionForm from './components/TransactionForm';
import TransactionList from './components/TransactionList';
import AIInsights from './components/AIInsights';
import BudgetTracker from './components/BudgetTracker';
import AccountSelector from './components/AccountSelector';
import { Transaction, TransactionType, CategoryBudget, Category, Account } from './types';
import { INITIAL_TRANSACTIONS, DEFAULT_ACCOUNTS } from './constants';

const App: React.FC = () => {
  const [transactions, setTransactions] = useState<Transaction[]>(() => {
    const saved = localStorage.getItem('fintrack_data');
    return saved ? JSON.parse(saved) : INITIAL_TRANSACTIONS;
  });

  const [budgets, setBudgets] = useState<CategoryBudget[]>(() => {
    const saved = localStorage.getItem('fintrack_budgets');
    return saved ? JSON.parse(saved) : [];
  });

  const [accounts, setAccounts] = useState<Account[]>(() => {
    const saved = localStorage.getItem('fintrack_accounts');
    return saved ? JSON.parse(saved) : DEFAULT_ACCOUNTS;
  });

  const [selectedAccountId, setSelectedAccountId] = useState<string | null>(null);

  useEffect(() => {
    localStorage.setItem('fintrack_data', JSON.stringify(transactions));
  }, [transactions]);

  useEffect(() => {
    localStorage.setItem('fintrack_budgets', JSON.stringify(budgets));
  }, [budgets]);

  useEffect(() => {
    localStorage.setItem('fintrack_accounts', JSON.stringify(accounts));
  }, [accounts]);

  const addTransaction = (newT: Omit<Transaction, 'id'>) => {
    const transaction: Transaction = {
      ...newT,
      id: Math.random().toString(36).substr(2, 9)
    };
    setTransactions(prev => [transaction, ...prev]);
  };

  const addAccount = (name: string, type: any) => {
    const newAccount: Account = {
      id: Math.random().toString(36).substr(2, 9),
      name,
      type,
      color: `#${Math.floor(Math.random()*16777215).toString(16)}`
    };
    setAccounts(prev => [...prev, newAccount]);
  };

  const deleteTransaction = (id: string) => {
    setTransactions(prev => prev.filter(t => t.id !== id));
  };

  const updateBudget = (category: Category, limit: number) => {
    setBudgets(prev => {
      const existing = prev.find(b => b.category === category);
      if (existing) {
        return prev.map(b => b.category === category ? { ...b, limit } : b);
      }
      return [...prev, { category, limit }];
    });
  };

  const filteredTransactions = useMemo(() => {
    if (!selectedAccountId) return transactions;
    return transactions.filter(t => t.accountId === selectedAccountId || t.toAccountId === selectedAccountId);
  }, [transactions, selectedAccountId]);

  const financialStatus = useMemo(() => {
    const relevantTransactions = filteredTransactions;
    
    // For specific accounts, we need to handle transfers differently in calculations
    const totalIncome = relevantTransactions.reduce((sum, t) => {
      // Income if it's type INCOME
      if (t.type === TransactionType.INCOME && (!selectedAccountId || t.accountId === selectedAccountId)) return sum + t.amount;
      // Also income for an account if it's the RECEIVER of a transfer
      if (t.type === TransactionType.TRANSFER && t.toAccountId === selectedAccountId) return sum + t.amount;
      return sum;
    }, 0);

    const totalExpenses = relevantTransactions.reduce((sum, t) => {
      // Expense if it's type EXPENSE
      if (t.type === TransactionType.EXPENSE && (!selectedAccountId || t.accountId === selectedAccountId)) return sum + t.amount;
      // Also expense for an account if it's the SENDER of a transfer
      if (t.type === TransactionType.TRANSFER && t.accountId === selectedAccountId) return sum + t.amount;
      return sum;
    }, 0);
    
    const totalBudget = budgets.reduce((sum, b) => sum + b.limit, 0);
    const budgetUsedPercent = totalBudget > 0 ? (totalExpenses / totalBudget) * 100 : 0;
    
    return {
      totalIncome,
      totalExpenses,
      totalBudget,
      budgetUsedPercent: Math.min(budgetUsedPercent, 100)
    };
  }, [filteredTransactions, budgets, selectedAccountId]);

  return (
    <div className="min-h-screen pb-12 bg-slate-50">
      <nav className="bg-white border-b border-slate-200 sticky top-0 z-50">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="flex justify-between h-16 items-center">
            <div className="flex items-center space-x-2">
              <div className="bg-indigo-600 p-2 rounded-lg shadow-indigo-200 shadow-md">
                <svg xmlns="http://www.w3.org/2000/svg" className="h-6 w-6 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 8c-1.657 0-3 .895-3 2s1.343 2 3 2 3 .895 3 2-1.343 2-3 2m0-8c1.11 0 2.08.402 2.599 1M12 8V7m0 1v8m0 0v1m0-1c-1.11 0-2.08-.402-2.599-1M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
                </svg>
              </div>
              <span className="text-xl font-black bg-gradient-to-r from-indigo-600 to-violet-600 bg-clip-text text-transparent">
                FinTrack AI
              </span>
            </div>
            <div className="flex items-center space-x-4">
               <button 
                onClick={() => {
                   if(confirm('Clear all data?')) {
                     setTransactions([]);
                     setBudgets([]);
                     setAccounts(DEFAULT_ACCOUNTS);
                     setSelectedAccountId(null);
                   }
                }}
                className="text-xs font-semibold text-slate-400 hover:text-red-500 uppercase tracking-wider transition-colors"
               >
                 Clear Database
               </button>
            </div>
          </div>
        </div>
      </nav>

      <main className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 pt-8">
        <div className="space-y-8">
          <AIInsights transactions={transactions} />

          <div className="grid grid-cols-1 lg:grid-cols-4 gap-8 items-start">
            <div className="lg:col-span-1 space-y-8 lg:sticky lg:top-24">
              <AccountSelector 
                accounts={accounts} 
                transactions={transactions} 
                onAddAccount={addAccount}
                selectedAccountId={selectedAccountId}
                onSelectAccount={setSelectedAccountId}
              />
              
              <TransactionForm onAdd={addTransaction} accounts={accounts} />
              
              <BudgetTracker 
                transactions={filteredTransactions} 
                budgets={budgets} 
                onUpdateBudget={updateBudget} 
              />
            </div>

            <div className="lg:col-span-3 space-y-8">
              <div className="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm">
                <div className="flex items-center justify-between mb-6">
                  <h2 className="text-xl font-bold text-slate-800">
                    {selectedAccountId 
                      ? accounts.find(a => a.id === selectedAccountId)?.name 
                      : 'Consolidated Overview'}
                  </h2>
                  <div className="flex items-center space-x-2 text-xs font-medium text-slate-400">
                    <span className="w-2 h-2 rounded-full bg-emerald-500"></span>
                    <span>Live Tracking</span>
                  </div>
                </div>
                <Dashboard transactions={filteredTransactions} />
              </div>
              
              <TransactionList transactions={filteredTransactions} onDelete={deleteTransaction} />
            </div>
          </div>
        </div>
      </main>

      <footer className="mt-20 border-t border-slate-200 py-12">
        <div className="max-w-7xl mx-auto px-4 flex flex-col items-center">
          <div className="flex items-center space-x-2 mb-4 grayscale opacity-50">
             <div className="bg-slate-600 p-1.5 rounded-md">
                <svg xmlns="http://www.w3.org/2000/svg" className="h-4 w-4 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 8c-1.657 0-3 .895-3 2s1.343 2 3 2 3 .895 3 2-1.343 2-3 2m0-8c1.11 0 2.08.402 2.599 1M12 8V7m0 1v8m0 0v1m0-1c-1.11 0-2.08-.402-2.599-1M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
                </svg>
              </div>
              <span className="text-sm font-bold text-slate-800 tracking-tight">FinTrack AI</span>
          </div>
          <p className="text-xs text-slate-400">
            Secure personal finance manager &bull; AI Powered Insights &bull; {new Date().getFullYear()}
          </p>
        </div>
      </footer>
    </div>
  );
};

export default App;
