# KasirAPP
Kasir_WarungKu
```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Aplikasi Kasir Sederhana</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        /* Custom scrollbar for a cleaner look */
        ::-webkit-scrollbar {
            width: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #f1f1f1; 
        }
        ::-webkit-scrollbar-thumb {
            background: #cbd5e1; 
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #94a3b8; 
        }
    </style>
</head>
<body class="bg-gray-100 font-sans h-screen flex flex-col">
    <!-- Header -->
    <header class="bg-blue-600 text-white p-4 shadow-md">
        <h1 class="text-2xl font-bold text-center"><i class="fas fa-cash-register mr-2"></i> KasirKita</h1>
    </header>

    <!-- Main Content -->
    <main class="flex-1 flex flex-col md:flex-row overflow-hidden max-w-7xl mx-auto w-full p-4 gap-4">
        
        <!-- Left Side: Menu Items -->
        <section class="flex-1 bg-white rounded-lg shadow-md flex flex-col overflow-hidden">
            <div class="p-4 bg-gray-50 border-b flex justify-between items-center">
                <h2 class="text-xl font-semibold text-gray-800">Daftar Menu</h2>
                <div class="relative">
                    <input type="text" id="searchInput" placeholder="Cari menu..." class="pl-8 pr-4 py-2 border rounded-full text-sm focus:outline-none focus:ring-2 focus:ring-blue-500 w-full sm:w-64">
                    <i class="fas fa-search absolute left-3 top-2.5 text-gray-400"></i>
                </div>
            </div>
            
            <div class="p-4 overflow-y-auto flex-1">
                <div id="menuContainer" class="grid grid-cols-2 sm:grid-cols-3 lg:grid-cols-4 gap-4">
                    <!-- Menu items will be injected here by JS -->
                </div>
            </div>
        </section>

        <!-- Right Side: Order Summary & Checkout -->
        <section class="w-full md:w-96 bg-white rounded-lg shadow-md flex flex-col overflow-hidden">
            <div class="p-4 bg-gray-50 border-b">
                <h2 class="text-xl font-semibold text-gray-800">Pesanan Saat Ini</h2>
            </div>
            
            <!-- Order Items List -->
            <div class="flex-1 overflow-y-auto p-4" id="orderList">
                <div id="emptyOrderMsg" class="text-center text-gray-500 mt-10">
                    <i class="fas fa-shopping-basket text-4xl mb-3 text-gray-300"></i>
                    <p>Belum ada pesanan.</p>
                </div>
                <!-- Order items will appear here -->
            </div>

            <!-- Checkout Area -->
            <div class="p-4 border-t bg-gray-50">
                <div class="flex justify-between items-center mb-2">
                    <span class="text-gray-600">Subtotal</span>
                    <span class="font-semibold" id="subtotalDisplay">Rp 0</span>
                </div>
                <div class="flex justify-between items-center mb-4 text-xl font-bold text-gray-800">
                    <span>Total</span>
                    <span id="totalDisplay">Rp 0</span>
                </div>
                
                <button id="payBtn" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-4 rounded-lg transition duration-200 disabled:opacity-50 disabled:cursor-not-allowed" disabled>
                    Bayar
                </button>
            </div>
        </section>
    </main>

    <!-- Payment Modal -->
    <div id="paymentModal" class="fixed inset-0 bg-black bg-opacity-50 hidden flex items-center justify-center z-50 p-4">
        <div class="bg-white rounded-xl shadow-2xl w-full max-w-md overflow-hidden transform transition-all">
            <div class="bg-blue-600 p-4 text-white flex justify-between items-center">
                <h3 class="text-xl font-bold">Proses Pembayaran</h3>
                <button id="closeModalBtn" class="text-white hover:text-gray-200">
                    <i class="fas fa-times text-xl"></i>
                </button>
            </div>
            <div class="p-6">
                <div class="text-center mb-6">
                    <p class="text-gray-600 mb-1">Total Tagihan</p>
                    <p class="text-3xl font-bold text-gray-800" id="modalTotalDisplay">Rp 0</p>
                </div>
                
                <div class="mb-4">
                    <label for="amountPaid" class="block text-sm font-medium text-gray-700 mb-1">Uang Diterima</label>
                    <div class="relative">
                        <span class="absolute inset-y-0 left-0 flex items-center pl-3 text-gray-500 font-semibold">Rp</span>
                        <input type="number" id="amountPaid" class="pl-10 w-full p-3 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 text-lg" placeholder="0">
                    </div>
                </div>

                <div class="mb-6 p-4 bg-gray-100 rounded-lg hidden" id="changeContainer">
                    <div class="flex justify-between items-center">
                        <span class="text-gray-600 font-medium">Kembalian</span>
                        <span class="text-2xl font-bold text-green-600" id="changeDisplay">Rp 0</span>
                    </div>
                </div>

                <button id="confirmPaymentBtn" class="w-full bg-green-500 hover:bg-green-600 text-white font-bold py-3 px-4 rounded-lg transition duration-200">
                    Selesai & Cetak Struk
                </button>
            </div>
        </div>
    </div>

    <!-- Notification Toast -->
    <div id="toast" class="fixed bottom-4 right-4 bg-green-500 text-white px-6 py-3 rounded-lg shadow-lg transform transition-all duration-300 translate-y-full opacity-0 z-50">
        Pembayaran berhasil!
    </div>

    <script>
        // Data Menu (Mocks a database)
        const menuItems = [
            { id: 1, name: 'Nasi Goreng Spesial', price: 25000, category: 'makanan', icon: 'fa-utensils', color: 'text-orange-500' },
            { id: 2, name: 'Mie Goreng Seafood', price: 30000, category: 'makanan', icon: 'fa-bowl-food', color: 'text-yellow-600' },
            { id: 3, name: 'Ayam Bakar Madu', price: 28000, category: 'makanan', icon: 'fa-drumstick-bite', color: 'text-red-500' },
            { id: 4, name: 'Sate Ayam (10 Tusuk)', price: 20000, category: 'makanan', icon: 'fa-utensils', color: 'text-orange-600' },
            { id: 5, name: 'Es Teh Manis', price: 5000, category: 'minuman', icon: 'fa-glass-water', color: 'text-amber-700' },
            { id: 6, name: 'Es Jeruk', price: 8000, category: 'minuman', icon: 'fa-martini-glass-citrus', color: 'text-orange-400' },
            { id: 7, name: 'Kopi Hitam', price: 10000, category: 'minuman', icon: 'fa-mug-hot', color: 'text-stone-800' },
            { id: 8, name: 'Jus Alpukat', price: 15000, category: 'minuman', icon: 'fa-blender', color: 'text-green-500' },
            { id: 9, name: 'Kentang Goreng', price: 12000, category: 'snack', icon: 'fa-pizza-slice', color: 'text-yellow-500' },
            { id: 10, name: 'Pisang Bakar Coklat', price: 15000, category: 'snack', icon: 'fa-cookie-bite', color: 'text-amber-800' }
        ];

        // State variables
        let cart = {}; // Store quantities by item ID
        let currentTotal = 0;

        // DOM Elements
        const menuContainer = document.getElementById('menuContainer');
        const searchInput = document.getElementById('searchInput');
        const orderList = document.getElementById('orderList');
        const emptyOrderMsg = document.getElementById('emptyOrderMsg');
        const subtotalDisplay = document.getElementById('subtotalDisplay');
        const totalDisplay = document.getElementById('totalDisplay');
        const payBtn = document.getElementById('payBtn');
        
        // Modal Elements
        const paymentModal = document.getElementById('paymentModal');
        const closeModalBtn = document.getElementById('closeModalBtn');
        const modalTotalDisplay = document.getElementById('modalTotalDisplay');
        const amountPaidInput = document.getElementById('amountPaid');
        const changeContainer = document.getElementById('changeContainer');
        const changeDisplay = document.getElementById('changeDisplay');
        const confirmPaymentBtn = document.getElementById('confirmPaymentBtn');
        const toast = document.getElementById('toast');

        // Helper to format currency
        const formatRupiah = (number) => {
            return new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0, maximumFractionDigits: 0 }).format(number);
        };

        // Render Menu Items
        function renderMenu(items) {
            menuContainer.innerHTML = '';
            if (items.length === 0) {
                menuContainer.innerHTML = '<p class="col-span-full text-center text-gray-500 mt-10">Menu tidak ditemukan.</p>';
                return;
            }

            items.forEach(item => {
                const count = cart[item.id] || 0;
                const card = document.createElement('div');
                card.className = `bg-white border rounded-xl p-4 flex flex-col items-center justify-between text-center cursor-pointer hover:shadow-md transition-shadow relative select-none ${count > 0 ? 'ring-2 ring-blue-500' : ''}`;
                
                // Add badge if item is in cart
                let badge = '';
                if (count > 0) {
                    badge = `<span class="absolute -top-2 -right-2 bg-blue-500 text-white text-xs font-bold w-6 h-6 rounded-full flex items-center justify-center">${count}</span>`;
                }

                card.innerHTML = `
                    ${badge}
                    <div class="w-16 h-16 rounded-full bg-gray-100 flex items-center justify-center mb-3 text-2xl ${item.color}">
                        <i class="fas ${item.icon}"></i>
                    </div>
                    <h3 class="font-medium text-sm text-gray-800 mb-1 leading-tight h-10 flex items-center">${item.name}</h3>
                    <p class="text-blue-600 font-bold text-sm mb-3">${formatRupiah(item.price)}</p>
                    <button class="w-full bg-blue-50 hover:bg-blue-100 text-blue-600 py-1.5 rounded-lg text-sm font-medium transition-colors" onclick="addToCart(${item.id})">
                        Tambah
                    </button>
                `;
                menuContainer.appendChild(card);
            });
        }

        // Cart Logic
        function addToCart(id) {
            if (cart[id]) {
                cart[id]++;
            } else {
                cart[id] = 1;
            }
            updateCart();
            // Re-render menu to show badges
            renderMenu(getFilteredMenu());
        }

        function removeFromCart(id) {
            if (cart[id]) {
                cart[id]--;
                if (cart[id] === 0) {
                    delete cart[id];
                }
            }
            updateCart();
            renderMenu(getFilteredMenu());
        }

        function deleteFromCart(id) {
            delete cart[id];
            updateCart();
            renderMenu(getFilteredMenu());
        }

        function updateCart() {
            orderList.innerHTML = '';
            currentTotal = 0;
            let totalItems = 0;

            const cartIds = Object.keys(cart);

            if (cartIds.length === 0) {
                orderList.appendChild(emptyOrderMsg);
                emptyOrderMsg.style.display = 'block';
                payBtn.disabled = true;
            } else {
                emptyOrderMsg.style.display = 'none';
                payBtn.disabled = false;

                cartIds.forEach(id => {
                    const item = menuItems.find(m => m.id == id);
                    const qty = cart[id];
                    const itemTotal = item.price * qty;
                    currentTotal += itemTotal;
                    totalItems += qty;

                    const orderItem = document.createElement('div');
                    orderItem.className = 'flex items-center justify-between mb-4 p-3 bg-white border rounded-lg shadow-sm';
                    orderItem.innerHTML = `
                        <div class="flex-1">
                            <h4 class="font-medium text-gray-800 text-sm">${item.name}</h4>
                            <p class="text-gray-500 text-xs">${formatRupiah(item.price)}</p>
                        </div>
                        <div class="flex items-center gap-3">
                            <div class="flex items-center bg-gray-100 rounded-lg">
                                <button onclick="removeFromCart(${item.id})" class="w-8 h-8 flex items-center justify-center text-gray-600 hover:text-blue-600 rounded-l-lg hover:bg-gray-200 transition-colors">
                                    <i class="fas fa-minus text-xs"></i>
                                </button>
                                <span class="w-8 text-center font-medium text-sm">${qty}</span>
                                <button onclick="addToCart(${item.id})" class="w-8 h-8 flex items-center justify-center text-gray-600 hover:text-blue-600 rounded-r-lg hover:bg-gray-200 transition-colors">
                                    <i class="fas fa-plus text-xs"></i>
                                </button>
                            </div>
                            <div class="text-right w-20 font-semibold text-sm">
                                ${formatRupiah(itemTotal)}
                            </div>
                        </div>
                    `;
                    orderList.appendChild(orderItem);
                });
            }

            // Update displays
            subtotalDisplay.innerText = formatRupiah(currentTotal);
            totalDisplay.innerText = formatRupiah(currentTotal);
            
            // Auto scroll to bottom of order list
            orderList.scrollTop = orderList.scrollHeight;
        }

        // Search functionality
        function getFilteredMenu() {
            const searchTerm = searchInput.value.toLowerCase();
            return menuItems.filter(item => item.name.toLowerCase().includes(searchTerm));
        }

        searchInput.addEventListener('input', () => {
            renderMenu(getFilteredMenu());
        });

        // Payment Modal Logic
        payBtn.addEventListener('click', () => {
            modalTotalDisplay.innerText = formatRupiah(currentTotal);
            amountPaidInput.value = '';
            changeContainer.classList.add('hidden');
            paymentModal.classList.remove('hidden');
            // Small delay to allow display before focusing
            setTimeout(() => amountPaidInput.focus(), 100);
        });

        closeModalBtn.addEventListener('click', () => {
            paymentModal.classList.add('hidden');
        });

        // Calculate change on input
        amountPaidInput.addEventListener('input', (e) => {
            const paid = parseFloat(e.target.value);
            if (!isNaN(paid) && paid >= currentTotal) {
                const change = paid - currentTotal;
                changeDisplay.innerText = formatRupiah(change);
                changeContainer.classList.remove('hidden');
                confirmPaymentBtn.disabled = false;
                confirmPaymentBtn.classList.remove('opacity-50', 'cursor-not-allowed');
            } else {
                changeContainer.classList.add('hidden');
                // Optional: disable confirm button if not enough money
                // confirmPaymentBtn.disabled = true;
                // confirmPaymentBtn.classList.add('opacity-50', 'cursor-not-allowed');
            }
        });

        confirmPaymentBtn.addEventListener('click', () => {
            const paid = parseFloat(amountPaidInput.value) || 0;
            if (paid < currentTotal) {
                // Simple custom alert instead of window.alert
                alertCustom('Uang yang dibayarkan kurang dari total tagihan.');
                return;
            }

            // Process successful payment
            paymentModal.classList.add('hidden');
            
            // Show toast notification
            toast.classList.remove('translate-y-full', 'opacity-0');
            setTimeout(() => {
                toast.classList.add('translate-y-full', 'opacity-0');
            }, 3000);

            // Reset cart
            cart = {};
            updateCart();
            renderMenu(menuItems);
            searchInput.value = '';
        });

        // Custom alert to avoid native browser alerts
        function alertCustom(message) {
            const alertDiv = document.createElement('div');
            alertDiv.className = 'fixed top-4 left-1/2 transform -translate-x-1/2 bg-red-500 text-white px-6 py-3 rounded-lg shadow-lg z-50 flex items-center gap-2 transition-all duration-300';
            alertDiv.innerHTML = `<i class="fas fa-exclamation-circle"></i> <span>${message}</span>`;
            document.body.appendChild(alertDiv);
            
            setTimeout(() => {
                alertDiv.style.opacity = '0';
                setTimeout(() => alertDiv.remove(), 300);
            }, 2000);
        }

        // Close modal when clicking outside
        window.addEventListener('click', (e) => {
            if (e.target === paymentModal) {
                paymentModal.classList.add('hidden');
            }
        });

        // Initialize the app
        renderMenu(menuItems);

    </script>
</body>
</html>

```
Jalankan script aplikasi di atas