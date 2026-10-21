<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistem Presensi Digital - SPENWAT SCOUT</title>
    
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        scout: {
                            50: '#FDF8F0',
                            100: '#F5E8D3',
                            200: '#E8CEA7',
                            500: '#8B5A2B',
                            600: '#6F421B',
                            700: '#5A3825',
                            800: '#3D200A',
                            900: '#261204',
                            gold: '#D4AF37',
                            amber: '#B8860B'
                        }
                    }
                }
            }
        }
    </script>
    
    <!-- Font Awesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <!-- HTML5 QR Code Scanner CDN -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html5-qrcode/2.3.8/html5-qrcode.min.js"></script>
    
    <!-- QRCode.js CDN for Generator -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700;800&display=swap');
        body {
            font-family: 'Plus Jakarta Sans', sans-serif;
            background-color: #FDF8F0;
        }
        
        .scout-gradient {
            background: linear-gradient(135deg, #3D200A 0%, #5A3825 50%, #8B5A2B 100%);
        }

        /* Printable Area Customization */
        @media print {
            body * {
                visibility: hidden;
            }
            #printableArea, #printableArea * {
                visibility: visible;
            }
            #printableArea {
                position: absolute;
                left: 0;
                top: 0;
                width: 100%;
            }
            .no-print {
                display: none !important;
            }
        }
    </style>
</head>
<body class="text-slate-800 antialiased min-h-screen flex flex-col justify-between">

    <!-- HEADER / BRANDING -->
    <header class="scout-gradient text-white shadow-lg border-b-4 border-amber-500 sticky top-0 z-40">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-3">
            <div class="flex flex-col sm:flex-row items-center justify-between gap-4">
                
                <!-- Logo and Title -->
                <div class="flex items-center space-x-4 text-center sm:text-left">
                    <img src="input_file_0.png" alt="Logo SPENWAT SCOUT" class="h-16 w-auto object-contain bg-white/10 p-1 rounded-xl backdrop-blur-sm border border-amber-400/40 shadow-md transition transform hover:scale-105" onerror="this.src='https://placehold.co/100x120/5A3825/D4AF37?text=SPENWAT+SCOUT'">
                    <div>
                        <div class="flex items-center justify-center sm:justify-start gap-2">
                            <h1 class="text-xl sm:text-2xl font-extrabold tracking-wide text-amber-300">SPENWAT SCOUT</h1>
                            <span class="bg-amber-500/30 text-amber-200 text-xs px-2 py-0.5 rounded border border-amber-400/30 font-semibold">PRESENSI DIGITAL</span>
                        </div>
                        <p class="text-xs sm:text-sm font-medium text-amber-100/90">Gugus Depan 10.005 & 10.006</p>
                        <p class="text-xs text-amber-200/70">UPT SMPN 1 WATANG PULU</p>
                    </div>
                </div>

                <!-- Clock & Status -->
                <div class="hidden md:flex flex-col items-end justify-center bg-black/20 px-4 py-2 rounded-xl border border-white/10">
                    <div id="liveClock" class="text-lg font-mono font-bold text-amber-300">00:00:00 WITA</div>
                    <div id="liveDate" class="text-xs text-slate-200">Rabu, 01 Januari 2025</div>
                </div>
            </div>
        </div>

        <!-- NAVIGATION TABS -->
        <div class="bg-black/30 backdrop-blur-md border-t border-white/10">
            <div class="max-w-7xl mx-auto px-2 sm:px-6 lg:px-8 flex overflow-x-auto space-x-1 sm:space-x-2 py-2 text-sm no-scrollbar">
                <button onclick="switchTab('scanner')" id="tab-scanner" class="nav-tab px-4 py-2 rounded-lg font-semibold flex items-center gap-2 whitespace-nowrap transition bg-amber-500 text-slate-950 shadow-md">
                    <i class="fa-solid fa-qrcode"></i> Pemindai Presensi
                </button>
                <button onclick="switchTab('members')" id="tab-members" class="nav-tab px-4 py-2 rounded-lg font-semibold flex items-center gap-2 whitespace-nowrap transition text-slate-200 hover:bg-white/10">
                    <i class="fa-solid fa-users"></i> Data Anggota & Kartu
                </button>
                <button onclick="switchTab('rekap')" id="tab-rekap" class="nav-tab px-4 py-2 rounded-lg font-semibold flex items-center gap-2 whitespace-nowrap transition text-slate-200 hover:bg-white/10">
                    <i class="fa-solid fa-clipboard-list"></i> Rekap Presensi
                </button>
                <button onclick="switchTab('settings')" id="tab-settings" class="nav-tab px-4 py-2 rounded-lg font-semibold flex items-center gap-2 whitespace-nowrap transition text-slate-200 hover:bg-white/10">
                    <i class="fa-solid fa-sliders"></i> Pengaturan
                </button>
            </div>
        </div>
    </header>

    <!-- MAIN CONTAINER -->
    <main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-6 flex-grow w-full">

        <!-- NOTIFICATION BANNER / TOAST CONTAINER -->
        <div id="toastContainer" class="fixed bottom-5 right-5 z-50 flex flex-col gap-2 pointer-events-none"></div>

        <!-- TAB 1: SCANNER & DASHBOARD -->
        <section id="sec-scanner" class="tab-content space-y-6">
            
            <!-- Quick Stats Dashboard -->
            <div class="grid grid-cols-2 md:grid-cols-5 gap-4">
                <div class="bg-white p-4 rounded-2xl shadow-sm border border-amber-100 flex items-center gap-4">
                    <div class="w-12 h-12 rounded-xl bg-amber-100 text-amber-800 flex items-center justify-center text-xl font-bold">
                        <i class="fa-solid fa-users"></i>
                    </div>
                    <div>
                        <p class="text-xs text-slate-500 font-medium">Total Anggota</p>
                        <h3 id="statTotalMembers" class="text-2xl font-bold text-slate-800">0</h3>
                    </div>
                </div>
                <div class="bg-white p-4 rounded-2xl shadow-sm border border-amber-100 flex items-center gap-4">
                    <div class="w-12 h-12 rounded-xl bg-emerald-100 text-emerald-700 flex items-center justify-center text-xl font-bold">
                        <i class="fa-solid fa-user-check"></i>
                    </div>
                    <div>
                        <p class="text-xs text-slate-500 font-medium">Hadir Hari Ini</p>
                        <h3 id="statHadirToday" class="text-2xl font-bold text-emerald-600">0</h3>
                    </div>
                </div>
                <div class="bg-white p-4 rounded-2xl shadow-sm border border-amber-100 flex items-center gap-4">
                    <div class="w-12 h-12 rounded-xl bg-blue-100 text-blue-700 flex items-center justify-center text-xl font-bold">
                        <i class="fa-solid fa-user-clock"></i>
                    </div>
                    <div>
                        <p class="text-xs text-slate-500 font-medium">Izin / Sakit</p>
                        <h3 id="statIzinToday" class="text-2xl font-bold text-blue-600">0</h3>
                    </div>
                </div>
                <div class="bg-white p-4 rounded-2xl shadow-sm border border-amber-100 flex items-center gap-4">
                    <div class="w-12 h-12 rounded-xl bg-rose-100 text-rose-700 flex items-center justify-center text-xl font-bold">
                        <i class="fa-solid fa-user-xmark"></i>
                    </div>
                    <div>
                        <p class="text-xs text-slate-500 font-medium">Alpha</p>
                        <h3 id="statAlphaToday" class="text-2xl font-bold text-rose-600">0</h3>
                    </div>
                </div>
                <div class="bg-white p-4 rounded-2xl shadow-sm border border-amber-100 flex items-center gap-4 col-span-2 md:col-span-1">
                    <div class="w-12 h-12 rounded-xl bg-amber-500/20 text-scout-700 flex items-center justify-center text-xl font-bold">
                        <i class="fa-solid fa-chart-pie"></i>
                    </div>
                    <div>
                        <p class="text-xs text-slate-500 font-medium">% Kehadiran</p>
                        <h3 id="statPercentToday" class="text-2xl font-bold text-scout-700">0%</h3>
                    </div>
                </div>
            </div>

            <!-- Scanner & Manual Input Grid -->
            <div class="grid grid-cols-1 lg:grid-cols-12 gap-6">
                
                <!-- Left Column: QR Scanner -->
                <div class="lg:col-span-7 bg-white rounded-2xl shadow-sm border border-amber-100 p-5 flex flex-col justify-between">
                    <div>
                        <div class="flex items-center justify-between pb-3 mb-4 border-b border-slate-100">
                            <h2 class="text-lg font-bold text-slate-800 flex items-center gap-2">
                                <i class="fa-solid fa-camera text-scout-600"></i> Pemindai QR / Barcode
                            </h2>
                            <span id="scanStatusBadge" class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-slate-100 text-slate-600">
                                Kamera Siap
                            </span>
                        </div>

                        <!-- Scanner Box -->
                        <div class="relative bg-slate-900 rounded-xl overflow-hidden min-h-[280px] flex items-center justify-center border-2 border-dashed border-amber-200">
                            <div id="reader" class="w-full h-full"></div>
                        </div>

                        <!-- Scanner Controls -->
                        <div class="mt-4 flex flex-wrap gap-2 justify-center">
                            <button id="btnStartScan" onclick="startScanner()" class="px-4 py-2 bg-emerald-600 hover:bg-emerald-700 text-white rounded-xl text-sm font-semibold transition flex items-center gap-2 shadow-sm">
                                <i class="fa-solid fa-play"></i> Mulai Kamera
                            </button>
                            <button id="btnStopScan" onclick="stopScanner()" class="px-4 py-2 bg-rose-600 hover:bg-rose-700 text-white rounded-xl text-sm font-semibold transition flex items-center gap-2 shadow-sm hidden">
                                <i class="fa-solid fa-stop"></i> Hentikan Kamera
                            </button>
                        </div>
                    </div>

                    <!-- Manual Fallback Input -->
                    <div class="mt-6 pt-4 border-t border-slate-100">
                        <label class="block text-xs font-semibold text-slate-500 uppercase tracking-wider mb-2">Input Manual ID / NTA Anggota</label>
                        <form onsubmit="handleManualSubmit(event)" class="flex gap-2">
                            <input type="text" id="manualInputId" placeholder="Contoh: SPW-10005-001" class="flex-1 px-4 py-2.5 rounded-xl border border-slate-300 focus:ring-2 focus:ring-amber-500 focus:border-amber-500 outline-none text-sm font-mono uppercase">
                            <button type="submit" class="px-5 py-2.5 bg-scout-700 hover:bg-scout-800 text-white font-semibold rounded-xl text-sm transition shadow-sm flex items-center gap-2">
                                <i class="fa-solid fa-paper-plane"></i> Proses
                            </button>
                        </form>
                    </div>
                </div>

                <!-- Right Column: Last Scanned Result & Live Log -->
                <div class="lg:col-span-5 flex flex-col gap-6">
                    
                    <!-- Scan Result Card -->
                    <div id="scanResultCard" class="bg-white rounded-2xl shadow-sm border border-amber-100 p-5 text-center flex flex-col items-center justify-center min-h-[220px]">
                        <div class="w-16 h-16 bg-amber-50 text-scout-600 rounded-full flex items-center justify-center text-2xl mb-3">
                            <i class="fa-solid fa-id-card"></i>
                        </div>
                        <h3 class="text-slate-600 font-medium text-sm">Menunggu Scan Anggota...</h3>
                        <p class="text-xs text-slate-400 mt-1 max-w-xs">Arahkan QR Code Kartu Anggota ke kamera atau ketik ID pada input manual.</p>
                    </div>

                    <!-- Today's Recent Scans -->
                    <div class="bg-white rounded-2xl shadow-sm border border-amber-100 p-5 flex-1 flex flex-col">
                        <div class="flex items-center justify-between pb-3 mb-3 border-b border-slate-100">
                            <h3 class="font-bold text-slate-800 text-sm flex items-center gap-2">
                                <i class="fa-solid fa-clock-rotate-left text-scout-600"></i> Pemindaian Terakhir
                            </h3>
                            <span id="recentCount" class="text-xs font-medium text-slate-500">0 Record</span>
                        </div>
                        <div id="recentList" class="space-y-2 overflow-y-auto max-h-[250px] pr-1 divide-y divide-slate-100">
                            <!-- Populated dynamically -->
                            <p class="text-center text-xs text-slate-400 py-6">Belum ada presensi tercatat hari ini.</p>
                        </div>
                    </div>

                </div>
            </div>
        </section>

        <!-- TAB 2: MEMBERS & QR CARD GENERATOR -->
        <section id="sec-members" class="tab-content space-y-6 hidden">
            
            <!-- Controls Bar -->
            <div class="bg-white p-4 rounded-2xl shadow-sm border border-amber-100 flex flex-col md:flex-row items-center justify-between gap-4">
                <div class="flex items-center gap-2 w-full md:w-auto">
                    <div class="relative w-full md:w-80">
                        <i class="fa-solid fa-magnifying-glass absolute left-3.5 top-3 text-slate-400 text-sm"></i>
                        <input type="text" id="searchMember" oninput="renderMembersTable()" placeholder="Cari nama, ID, kelas/jabatan..." class="w-full pl-10 pr-4 py-2 border border-slate-300 rounded-xl text-sm focus:ring-2 focus:ring-amber-500 outline-none">
                    </div>
                </div>
                <div class="flex items-center gap-2 w-full md:w-auto justify-end">
                    <button onclick="openAddMemberModal()" class="px-4 py-2 bg-emerald-600 hover:bg-emerald-700 text-white font-semibold rounded-xl text-sm transition flex items-center gap-2 shadow-sm w-full md:w-auto justify-center">
                        <i class="fa-solid fa-user-plus"></i> Tambah Anggota
                    </button>
                </div>
            </div>

            <!-- Members Table -->
            <div class="bg-white rounded-2xl shadow-sm border border-amber-100 overflow-hidden">
                <div class="overflow-x-auto">
                    <table class="w-full text-left border-collapse">
                        <thead>
                            <tr class="bg-slate-50 border-b border-slate-200 text-xs font-bold text-slate-600 uppercase tracking-wider">
                                <th class="p-4">ID / NTA</th>
                                <th class="p-4">Nama Lengkap</th>
                                <th class="p-4">Kelas / Jabatan</th>
                                <th class="p-4">Tingkat</th>
                                <th class="p-4">Gender</th>
                                <th class="p-4 text-center">Aksi</th>
                            </tr>
                        </thead>
                        <tbody id="membersTableBody" class="divide-y divide-slate-100 text-sm">
                            <!-- Populated dynamically -->
                        </tbody>
                    </table>
                </div>
            </div>
        </section>

        <!-- TAB 3: REKAP PRESENSI -->
        <section id="sec-rekap" class="tab-content space-y-6 hidden">
            
            <!-- Filter & Export Controls -->
            <div class="bg-white p-4 rounded-2xl shadow-sm border border-amber-100 flex flex-col lg:flex-row items-center justify-between gap-4">
                <div class="grid grid-cols-1 sm:grid-cols-3 gap-3 w-full lg:w-auto">
                    <div>
                        <label class="block text-xs font-semibold text-slate-500 mb-1">Tanggal</label>
                        <input type="date" id="rekapFilterDate" onchange="renderRekapTable()" class="w-full px-3 py-2 border border-slate-300 rounded-xl text-sm focus:ring-2 focus:ring-amber-500 outline-none">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-500 mb-1">Cari Nama / ID</label>
                        <input type="text" id="rekapSearch" oninput="renderRekapTable()" placeholder="Ketik kata kunci..." class="w-full px-3 py-2 border border-slate-300 rounded-xl text-sm focus:ring-2 focus:ring-amber-500 outline-none">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-500 mb-1">Status Kehadiran</label>
                        <select id="rekapFilterStatus" onchange="renderRekapTable()" class="w-full px-3 py-2 border border-slate-300 rounded-xl text-sm focus:ring-2 focus:ring-amber-500 outline-none">
                            <option value="ALL">Semua Status</option>
                            <option value="Hadir">Hadir</option>
                            <option value="Izin">Izin</option>
                            <option value="Sakit">Sakit</option>
                            <option value="Alpha">Alpha</option>
                        </select>
                    </div>
                </div>

                <div class="flex items-center gap-2 w-full lg:w-auto justify-end">
                    <button onclick="exportToCSV()" class="px-4 py-2 bg-emerald-600 hover:bg-emerald-700 text-white font-semibold rounded-xl text-sm transition flex items-center gap-2 shadow-sm">
                        <i class="fa-solid fa-file-excel"></i> Ekspor CSV
                    </button>
                    <button onclick="printRekapReport()" class="px-4 py-2 bg-scout-800 hover:bg-scout-900 text-white font-semibold rounded-xl text-sm transition flex items-center gap-2 shadow-sm">
                        <i class="fa-solid fa-print"></i> Cetak Laporan
                    </button>
                </div>
            </div>

            <!-- Rekap Table Area -->
            <div id="printableArea" class="bg-white rounded-2xl shadow-sm border border-amber-100 overflow-hidden p-4">
                
                <!-- Print Header (Visible during print) -->
                <div class="hidden print:block text-center mb-6 pb-4 border-b-2 border-slate-800">
                    <div class="flex items-center justify-center gap-4 mb-2">
                        <img src="input_file_0.png" class="h-16 w-auto" onerror="this.src='https://placehold.co/100x120/5A3825/D4AF37?text=SPENWAT+SCOUT'">
                        <div>
                            <h2 class="text-xl font-bold uppercase text-slate-900">Laporan Presensi Digital Anggota Pramuka</h2>
                            <h3 class="text-base font-semibold text-slate-700">SPENWAT SCOUT - Gudep 10.005 & 10.006</h3>
                            <p class="text-xs text-slate-500">UPT SMPN 1 WATANG PULU</p>
                        </div>
                    </div>
                    <p id="printMeta" class="text-xs font-mono text-slate-600 mt-2">Tanggal: -</p>
                </div>

                <div class="overflow-x-auto">
                    <table class="w-full text-left border-collapse">
                        <thead>
                            <tr class="bg-slate-50 border-b border-slate-200 text-xs font-bold text-slate-600 uppercase tracking-wider">
                                <th class="p-3">Waktu</th>
                                <th class="p-3">ID / NTA</th>
                                <th class="p-3">Nama Anggota</th>
                                <th class="p-3">Kelas / Jabatan</th>
                                <th class="p-3">Tingkat</th>
                                <th class="p-3 text-center">Status</th>
                                <th class="p-3 text-center no-print">Aksi</th>
                            </tr>
                        </thead>
                        <tbody id="rekapTableBody" class="divide-y divide-slate-100 text-sm">
                            <!-- Populated dynamically -->
                        </tbody>
                    </table>
                </div>
            </div>
        </section>

        <!-- TAB 4: SETTINGS & DATA MANAGEMENT -->
        <section id="sec-settings" class="tab-content space-y-6 hidden">
            <div class="bg-white rounded-2xl shadow-sm border border-amber-100 p-6 max-w-2xl mx-auto space-y-6">
                <div>
                    <h3 class="text-lg font-bold text-slate-800 mb-1">Pengaturan & Manajemen Data</h3>
                    <p class="text-sm text-slate-500">Kelola penyimpanan data lokal dan konfigurasi presensi SPENWAT SCOUT.</p>
                </div>

                <hr class="border-slate-100">

                <div class="space-y-4">
                    <div class="flex items-center justify-between p-4 bg-amber-50/50 rounded-xl border border-amber-200">
                        <div>
                            <h4 class="font-semibold text-slate-800 text-sm">Muat Ulang Data Sampel (Demo)</h4>
                            <p class="text-xs text-slate-500">Isi ulang data anggota dan contoh presensi default.</p>
                        </div>
                        <button onclick="resetDemoData()" class="px-4 py-2 bg-amber-600 hover:bg-amber-700 text-white font-semibold rounded-xl text-xs transition shadow-sm">
                            Reset Demo
                        </button>
                    </div>

                    <div class="flex items-center justify-between p-4 bg-rose-50 rounded-xl border border-rose-100">
                        <div>
                            <h4 class="font-semibold text-rose-800 text-sm">Hapus Semua Data Presensi</h4>
                            <p class="text-xs text-rose-600">Bersihkan seluruh catatan log presensi yang tersimpan.</p>
                        </div>
                        <button onclick="clearAttendanceLogs()" class="px-4 py-2 bg-rose-600 hover:bg-rose-700 text-white font-semibold rounded-xl text-xs transition shadow-sm">
                            Hapus Log
                        </button>
                    </div>
                </div>
            </div>
        </section>

    </main>

    <!-- FOOTER -->
    <footer class="bg-slate-900 text-slate-400 text-xs py-4 border-t border-slate-800 text-center">
        <div class="max-w-7xl mx-auto px-4">
            <p>© 2025 SPENWAT SCOUT - Gudep 10.005 & 10.006 UPT SMPN 1 Watang Pulu. All rights reserved.</p>
        </div>
    </footer>

    <!-- MODAL: MEMBER FORM (ADD/EDIT) -->
    <div id="modalMemberForm" class="fixed inset-0 z-50 bg-black/60 backdrop-blur-sm hidden flex items-center justify-center p-4">
        <div class="bg-white rounded-2xl shadow-xl w-full max-w-md overflow-hidden transform transition-all">
            <div class="scout-gradient text-white p-4 flex items-center justify-between">
                <h3 id="modalMemberTitle" class="font-bold text-amber-300">Tambah Anggota Baru</h3>
                <button onclick="closeMemberModal()" class="text-white/80 hover:text-white text-lg"><i class="fa-solid fa-xmark"></i></button>
            </div>
            <form onsubmit="saveMember(event)" class="p-5 space-y-4">
                <input type="hidden" id="formMemberOriginalId">
                
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">ID Anggota / NTA *</label>
                    <input type="text" id="formMemberId" required placeholder="SPW-10005-001" class="w-full px-3 py-2 border border-slate-300 rounded-xl text-sm focus:ring-2 focus:ring-amber-500 outline-none font-mono">
                </div>

                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">Nama Lengkap *</label>
                    <input type="text" id="formMemberName" required placeholder="Ahmad Rizky" class="w-full px-3 py-2 border border-slate-300 rounded-xl text-sm focus:ring-2 focus:ring-amber-500 outline-none">
                </div>

                <div class="grid grid-cols-2 gap-3">
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">Kelas / Jabatan *</label>
                        <input type="text" id="formMemberKelas" required placeholder="Kelas 7A / Pratama" class="w-full px-3 py-2 border border-slate-300 rounded-xl text-sm focus:ring-2 focus:ring-amber-500 outline-none">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">Tingkat *</label>
                        <select id="formMemberTingkat" class="w-full px-3 py-2 border border-slate-300 rounded-xl text-sm focus:ring-2 focus:ring-amber-500 outline-none">
                            <option value="Penggalang Ramu">Penggalang Ramu</option>
                            <option value="Penggalang Rakit">Penggalang Rakit</option>
                            <option value="Penggalang Terap">Penggalang Terap</option>
                            <option value="Penggalang Garuda">Penggalang Garuda</option>
                            <option value="Pembina">Pembina / Pendamping</option>
                        </select>
                    </div>
                </div>

                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">Jenis Kelamin *</label>
                    <select id="formMemberGender" class="w-full px-3 py-2 border border-slate-300 rounded-xl text-sm focus:ring-2 focus:ring-amber-500 outline-none">
                        <option value="L">Laki-Laki (Putra)</option>
                        <option value="P">Perempuan (Putri)</option>
                    </select>
                </div>

                <div class="pt-3 flex gap-2 justify-end">
                    <button type="button" onclick="closeMemberModal()" class="px-4 py-2 border border-slate-300 rounded-xl text-xs font-semibold text-slate-600 hover:bg-slate-50">Batal</button>
                    <button type="submit" class="px-4 py-2 bg-scout-700 hover:bg-scout-800 text-white font-semibold rounded-xl text-xs shadow-sm">Simpan Data</button>
                </div>
            </form>
        </div>
    </div>

    <!-- MODAL: MANUAL STATUS ENTRY -->
    <div id="modalStatusForm" class="fixed inset-0 z-50 bg-black/60 backdrop-blur-sm hidden flex items-center justify-center p-4">
        <div class="bg-white rounded-2xl shadow-xl w-full max-w-sm overflow-hidden">
            <div class="bg-scout-800 text-white p-4 flex items-center justify-between">
                <h3 class="font-bold text-amber-300 text-sm">Pilih Status Presensi Manual</h3>
                <button onclick="closeStatusModal()" class="text-white/80 hover:text-white text-lg"><i class="fa-solid fa-xmark"></i></button>
            </div>
            <div class="p-5 space-y-4">
                <p id="statusMemberName" class="font-bold text-slate-800 text-center text-base"></p>
                <div class="grid grid-cols-2 gap-2">
                    <button onclick="confirmManualStatus('Hadir')" class="p-3 bg-emerald-100 hover:bg-emerald-200 text-emerald-800 font-bold rounded-xl text-sm flex flex-col items-center justify-center gap-1">
                        <i class="fa-solid fa-circle-check text-lg"></i> Hadir
                    </button>
                    <button onclick="confirmManualStatus('Izin')" class="p-3 bg-blue-100 hover:bg-blue-200 text-blue-800 font-bold rounded-xl text-sm flex flex-col items-center justify-center gap-1">
                        <i class="fa-solid fa-envelope text-lg"></i> Izin
                    </button>
                    <button onclick="confirmManualStatus('Sakit')" class="p-3 bg-amber-100 hover:bg-amber-200 text-amber-800 font-bold rounded-xl text-sm flex flex-col items-center justify-center gap-1">
                        <i class="fa-solid fa-notes-medical text-lg"></i> Sakit
                    </button>
                    <button onclick="confirmManualStatus('Alpha')" class="p-3 bg-rose-100 hover:bg-rose-200 text-rose-800 font-bold rounded-xl text-sm flex flex-col items-center justify-center gap-1">
                        <i class="fa-solid fa-circle-xmark text-lg"></i> Alpha
                    </button>
                </div>
            </div>
        </div>
    </div>

    <!-- MODAL: QR CARD PREVIEW & PRINT -->
    <div id="modalQrCard" class="fixed inset-0 z-50 bg-black/60 backdrop-blur-sm hidden flex items-center justify-center p-4">
        <div class="bg-white rounded-2xl shadow-xl w-full max-w-sm overflow-hidden text-center">
            <div class="p-4 border-b border-slate-100 flex justify-between items-center bg-slate-50">
                <h3 class="font-bold text-slate-800 text-sm">Kartu Digital Anggota</h3>
                <button onclick="closeQrModal()" class="text-slate-400 hover:text-slate-600"><i class="fa-solid fa-xmark"></i></button>
            </div>
            
            <div class="p-6 flex flex-col items-center" id="cardPrintArea">
                <!-- ID Card Container -->
                <div class="w-full bg-gradient-to-br from-scout-900 via-scout-800 to-scout-700 rounded-2xl text-white p-5 shadow-lg border-2 border-amber-400 relative overflow-hidden">
                    
                    <!-- Card Background Motif -->
                    <div class="absolute -right-10 -bottom-10 opacity-10 pointer-events-none">
                        <i class="fa-solid fa-shield-halved text-9xl"></i>
                    </div>

                    <!-- Header -->
                    <div class="flex items-center gap-3 border-b border-amber-400/30 pb-3 mb-4 text-left">
                        <img src="input_file_0.png" class="h-10 w-auto bg-white/10 p-0.5 rounded-lg" onerror="this.src='https://placehold.co/80x80/5A3825/D4AF37?text=SCOUT'">
                        <div>
                            <h4 class="text-xs font-extrabold text-amber-300 tracking-wider">SPENWAT SCOUT</h4>
                            <p class="text-[10px] text-amber-100/80">GUDEP 10.005 & 10.006</p>
                        </div>
                    </div>

                    <!-- Body Info -->
                    <div class="text-left mb-4 space-y-1">
                        <p id="cardMemberName" class="font-bold text-base text-amber-200 truncate">Nama Anggota</p>
                        <p id="cardMemberId" class="text-xs font-mono text-slate-300">SPW-000</p>
                        <div class="flex gap-2 text-[11px] text-amber-100/90 pt-1">
                            <span id="cardMemberKelas" class="bg-white/10 px-2 py-0.5 rounded">Kelas: -</span>
                            <span id="cardMemberTingkat" class="bg-white/10 px-2 py-0.5 rounded">Penggalang</span>
                        </div>
                    </div>

                    <!-- QR Display Box -->
                    <div class="bg-white p-3 rounded-xl shadow-inner flex justify-center items-center my-2">
                        <div id="qrcodeDisplay" class="flex justify-center"></div>
                    </div>
                    
                    <p class="text-[9px] text-amber-200/60 mt-2">Kartu Tanda Anggota Resmi Presensi Digital</p>
                </div>
            </div>

            <div class="p-4 bg-slate-50 border-t border-slate-100 flex justify-center gap-2">
                <button onclick="printCardOnly()" class="px-4 py-2 bg-scout-700 text-white rounded-xl text-xs font-semibold hover:bg-scout-800 transition flex items-center gap-1.5 shadow-sm">
                    <i class="fa-solid fa-print"></i> Cetak Kartu
                </button>
            </div>
        </div>
    </div>

    <!-- AUDIO SOUND EFFECTS FOR SCAN -->
    <script>
        const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        function playBeep(success = true) {
            if (audioCtx.state === 'suspended') {
                audioCtx.resume();
            }
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            osc.connect(gain);
            gain.connect(audioCtx.destination);
            
            if (success) {
                osc.type = 'sine';
                osc.frequency.setValueAtTime(880, audioCtx.currentTime);
                gain.gain.setValueAtTime(0.1, audioCtx.currentTime);
                osc.start();
                osc.stop(audioCtx.currentTime + 0.15);
            } else {
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(220, audioCtx.currentTime);
                gain.gain.setValueAtTime(0.2, audioCtx.currentTime);
                osc.start();
                osc.stop(audioCtx.currentTime + 0.3);
            }
        }
    </script>

    <!-- MAIN APP JAVASCRIPT LOGIC -->
    <script>
        // DEFAULT DEMO MEMBERS (WITHOUT REGU)
        const DEFAULT_MEMBERS = [
            { id: "SPW-10005-001", name: "Ahmad Fauzi", kelas: "Kelas 8A", tingkat: "Penggalang Ramu", gender: "L" },
            { id: "SPW-10005-002", name: "Budi Santoso", kelas: "Kelas 8B", tingkat: "Penggalang Rakit", gender: "L" },
            { id: "SPW-10006-001", name: "Siti Rahma", kelas: "Kelas 7A", tingkat: "Penggalang Ramu", gender: "P" },
            { id: "SPW-10006-002", name: "Nur Aini", kelas: "Kelas 9C", tingkat: "Penggalang Terap", gender: "P" },
            { id: "SPW-10005-003", name: "Muhammad Rizky", kelas: "Kelas 8C", tingkat: "Penggalang Rakit", gender: "L" },
            { id: "SPW-10006-003", name: "Dewi Lestari", kelas: "Kelas 7B", tingkat: "Penggalang Ramu", gender: "P" }
        ];

        // STATE MANAGEMENT
        let members = JSON.parse(localStorage.getItem('spenwat_members_v2')) || DEFAULT_MEMBERS;
        let attendanceLogs = JSON.parse(localStorage.getItem('spenwat_logs_v2')) || [];
        let html5QrCode = null;
        let isScanning = false;
        let scanCooldown = false;
        let pendingMemberId = null;

        // ON APP LOAD
        window.addEventListener('DOMContentLoaded', () => {
            initClock();
            saveData();
            updateStats();
            renderRecentScans();
            
            // Set default date for rekap filter
            document.getElementById('rekapFilterDate').valueAsDate = new Date();
        });

        // REAL-TIME CLOCK
        function initClock() {
            function update() {
                const now = new Date();
                document.getElementById('liveClock').textContent = now.toLocaleTimeString('id-ID') + " WITA";
                document.getElementById('liveDate').textContent = now.toLocaleDateString('id-ID', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' });
            }
            update();
            setInterval(update, 1000);
        }

        // PERSISTENCE
        function saveData() {
            localStorage.setItem('spenwat_members_v2', JSON.stringify(members));
            localStorage.setItem('spenwat_logs_v2', JSON.stringify(attendanceLogs));
        }

        // NAVIGATION TAB SWITCHER
        function switchTab(tabName) {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
            document.querySelectorAll('.nav-tab').forEach(el => {
                el.classList.remove('bg-amber-500', 'text-slate-950', 'shadow-md');
                el.classList.add('text-slate-200', 'hover:bg-white/10');
            });

            document.getElementById(`sec-${tabName}`).classList.remove('hidden');
            const activeNav = document.getElementById(`tab-${tabName}`);
            activeNav.classList.add('bg-amber-500', 'text-slate-950', 'shadow-md');
            activeNav.classList.remove('text-slate-200', 'hover:bg-white/10');

            if (tabName === 'members') renderMembersTable();
            if (tabName === 'rekap') renderRekapTable();
            
            if (tabName !== 'scanner' && isScanning) {
                stopScanner();
            }
        }

        // =========================================================
        // QR CODE SCANNER ENGINE
        // =========================================================
        function startScanner() {
            if (isScanning) return;

            html5QrCode = new Html5Qrcode("reader");
            const config = { fps: 10, qrbox: { width: 220, height: 220 } };

            html5QrCode.start(
                { facingMode: "environment" },
                config,
                onScanSuccess,
                onScanFailure
            ).then(() => {
                isScanning = true;
                document.getElementById('btnStartScan').classList.add('hidden');
                document.getElementById('btnStopScan').classList.remove('hidden');
                
                const badge = document.getElementById('scanStatusBadge');
                badge.className = "inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-emerald-100 text-emerald-800 animate-pulse";
                badge.innerHTML = `<span class="w-2 h-2 mr-1.5 bg-emerald-500 rounded-full"></span> Kamera Aktif`;
            }).catch(err => {
                showToast("Gagal mengakses kamera: " + err, "error");
            });
        }

        function stopScanner() {
            if (html5QrCode && isScanning) {
                html5QrCode.stop().then(() => {
                    isScanning = false;
                    document.getElementById('btnStartScan').classList.remove('hidden');
                    document.getElementById('btnStopScan').classList.add('hidden');
                    
                    const badge = document.getElementById('scanStatusBadge');
                    badge.className = "inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-slate-100 text-slate-600";
                    badge.innerHTML = "Kamera Siap";
                }).catch(err => console.error(err));
            }
        }

        function onScanSuccess(decodedText) {
            if (scanCooldown) return;
            scanCooldown = true;
            setTimeout(() => { scanCooldown = false; }, 2500);

            processAttendance(decodedText.trim(), "Hadir");
        }

        function onScanFailure(error) {
            // Silently ignore frame failure
        }

        function handleManualSubmit(e) {
            e.preventDefault();
            const input = document.getElementById('manualInputId');
            const val = input.value.trim().toUpperCase();
            if (val) {
                const member = members.find(m => m.id.toUpperCase() === val);
                if (member) {
                    pendingMemberId = member.id;
                    document.getElementById('statusMemberName').textContent = `${member.name} (${member.id})`;
                    document.getElementById('modalStatusForm').classList.remove('hidden');
                } else {
                    processAttendance(val, "Hadir");
                }
                input.value = '';
            }
        }

        function closeStatusModal() {
            document.getElementById('modalStatusForm').classList.add('hidden');
            pendingMemberId = null;
        }

        function confirmManualStatus(status) {
            if (pendingMemberId) {
                processAttendance(pendingMemberId, status);
                closeStatusModal();
            }
        }

        // PROCESS ATTENDANCE LOGIC
        function processAttendance(memberId, statusChoice = "Hadir") {
            const member = members.find(m => m.id.toUpperCase() === memberId.toUpperCase());

            if (!member) {
                playBeep(false);
                showToast(`Anggota dengan ID "${memberId}" tidak ditemukan!`, "error");
                renderScanCard(null, memberId);
                return;
            }

            const todayStr = new Date().toISOString().split('T')[0];
            const existingLog = attendanceLogs.find(l => l.memberId === member.id && l.date === todayStr);

            if (existingLog) {
                playBeep(false);
                showToast(`${member.name} sudah tercatat presensi hari ini (${existingLog.time})!`, "warning");
                renderScanCard(member, member.id, "SUDAH_PRESENSI", existingLog);
                return;
            }

            // Create new record
            const now = new Date();
            const newLog = {
                id: Date.now(),
                memberId: member.id,
                memberName: member.name,
                kelas: member.kelas,
                tingkat: member.tingkat,
                date: todayStr,
                time: now.toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' }),
                timestamp: now.getTime(),
                status: statusChoice
            };

            attendanceLogs.unshift(newLog);
            saveData();
            playBeep(statusChoice === "Hadir");
            showToast(`Presensi ${statusChoice}: ${member.name}`, "success");

            renderScanCard(member, member.id, "BERHASIL", newLog);
            updateStats();
            renderRecentScans();
        }

        function renderScanCard(member, id, status = "NOT_FOUND", log = null) {
            const container = document.getElementById('scanResultCard');
            if (!member) {
                container.innerHTML = `
                    <div class="w-16 h-16 bg-rose-100 text-rose-600 rounded-full flex items-center justify-center text-2xl mb-3">
                        <i class="fa-solid fa-user-xmark"></i>
                    </div>
                    <h3 class="text-rose-600 font-bold text-base">Anggota Tidak Ditemukan</h3>
                    <p class="text-xs font-mono text-slate-500 mt-1">ID: ${id}</p>
                    <p class="text-xs text-slate-400 mt-2">Pastikan ID telah terdaftar di Data Anggota.</p>
                `;
                return;
            }

            const isAlready = status === "SUDAH_PRESENSI";
            const badgeBg = isAlready ? "bg-amber-100 text-amber-800 border-amber-300" : "bg-emerald-100 text-emerald-800 border-emerald-300";
            const badgeText = isAlready ? "SUDAH TERCATAT" : `STATUS: ${(log ? log.status : 'HADIR').toUpperCase()}`;

            container.innerHTML = `
                <div class="w-full bg-slate-50 rounded-xl p-4 border border-slate-200 text-left relative overflow-hidden">
                    <div class="flex items-center justify-between mb-3 pb-2 border-b border-slate-200">
                        <span class="text-[11px] font-bold px-2.5 py-0.5 rounded-full border ${badgeBg}">
                            ${badgeText}
                        </span>
                        <span class="text-xs font-mono text-slate-500">${log ? log.time : ''} WITA</span>
                    </div>
                    <h3 class="font-bold text-slate-800 text-lg">${member.name}</h3>
                    <p class="text-xs font-mono text-slate-500 mb-3">${member.id}</p>
                    <div class="flex gap-2 text-xs">
                        <span class="bg-amber-100 text-amber-800 px-2 py-1 rounded-md font-medium">${member.kelas}</span>
                        <span class="bg-slate-200 text-slate-700 px-2 py-1 rounded-md">${member.tingkat}</span>
                    </div>
                </div>
            `;
        }

        function renderRecentScans() {
            const listContainer = document.getElementById('recentList');
            const countLabel = document.getElementById('recentCount');
            const todayStr = new Date().toISOString().split('T')[0];

            const todayLogs = attendanceLogs.filter(l => l.date === todayStr);
            countLabel.textContent = `${todayLogs.length} Record`;

            if (todayLogs.length === 0) {
                listContainer.innerHTML = `<p class="text-center text-xs text-slate-400 py-6">Belum ada presensi tercatat hari ini.</p>`;
                return;
            }

            listContainer.innerHTML = todayLogs.map(l => {
                let badgeClass = "bg-emerald-100 text-emerald-800";
                if (l.status === "Izin") badgeClass = "bg-blue-100 text-blue-800";
                if (l.status === "Sakit") badgeClass = "bg-amber-100 text-amber-800";
                if (l.status === "Alpha") badgeClass = "bg-rose-100 text-rose-800";

                return `
                    <div class="py-2.5 flex items-center justify-between text-xs">
                        <div>
                            <p class="font-bold text-slate-800">${l.memberName}</p>
                            <p class="text-[11px] text-slate-500">${l.kelas} • ${l.memberId}</p>
                        </div>
                        <div class="text-right">
                            <span class="inline-block px-2 py-0.5 rounded ${badgeClass} font-semibold text-[10px]">${l.status}</span>
                            <p class="text-[10px] text-slate-400 font-mono mt-0.5">${l.time}</p>
                        </div>
                    </div>
                `;
            }).join('');
        }

        // DASHBOARD STATS CALCULATOR
        function updateStats() {
            const todayStr = new Date().toISOString().split('T')[0];
            const totalMembers = members.length;
            const todayLogs = attendanceLogs.filter(l => l.date === todayStr);
            
            const hadir = todayLogs.filter(l => l.status === "Hadir").length;
            const izinSakit = todayLogs.filter(l => l.status === "Izin" || l.status === "Sakit").length;
            const alpha = todayLogs.filter(l => l.status === "Alpha").length;
            const percentage = totalMembers > 0 ? Math.round((hadir / totalMembers) * 100) : 0;

            document.getElementById('statTotalMembers').textContent = totalMembers;
            document.getElementById('statHadirToday').textContent = hadir;
            document.getElementById('statIzinToday').textContent = izinSakit;
            document.getElementById('statAlphaToday').textContent = alpha;
            document.getElementById('statPercentToday').textContent = `${percentage}%`;
        }

        // =========================================================
        // MEMBER MANAGEMENT & QR CARD GENERATOR
        // =========================================================
        function renderMembersTable() {
            const tbody = document.getElementById('membersTableBody');
            const searchQuery = document.getElementById('searchMember').value.toLowerCase();

            const filtered = members.filter(m => {
                return m.name.toLowerCase().includes(searchQuery) ||
                       m.id.toLowerCase().includes(searchQuery) ||
                       m.kelas.toLowerCase().includes(searchQuery) ||
                       m.tingkat.toLowerCase().includes(searchQuery);
            });

            if (filtered.length === 0) {
                tbody.innerHTML = `<tr><td colspan="6" class="text-center py-6 text-slate-400 text-xs">Tidak ada data anggota ditemukan.</td></tr>`;
                return;
            }

            tbody.innerHTML = filtered.map(m => `
                <tr class="hover:bg-slate-50 transition">
                    <td class="p-4 font-mono font-semibold text-scout-700">${m.id}</td>
                    <td class="p-4 font-bold text-slate-800">${m.name}</td>
                    <td class="p-4"><span class="bg-amber-100 text-amber-800 text-xs font-semibold px-2.5 py-1 rounded-md">${m.kelas}</span></td>
                    <td class="p-4 text-slate-600">${m.tingkat}</td>
                    <td class="p-4"><span class="text-xs px-2 py-0.5 rounded ${m.gender === 'L' ? 'bg-blue-100 text-blue-700' : 'bg-pink-100 text-pink-700'}">${m.gender === 'L' ? 'Putra' : 'Putri'}</span></td>
                    <td class="p-4 text-center">
                        <div class="flex items-center justify-center gap-1.5">
                            <button onclick="openQrModal('${m.id}')" title="Kartu QR" class="p-2 bg-slate-100 hover:bg-amber-500 hover:text-white rounded-lg text-slate-600 text-xs transition">
                                <i class="fa-solid fa-qrcode"></i>
                            </button>
                            <button onclick="editMember('${m.id}')" title="Edit" class="p-2 bg-slate-100 hover:bg-blue-600 hover:text-white rounded-lg text-slate-600 text-xs transition">
                                <i class="fa-solid fa-pen"></i>
                            </button>
                            <button onclick="deleteMember('${m.id}')" title="Hapus" class="p-2 bg-slate-100 hover:bg-rose-600 hover:text-white rounded-lg text-slate-600 text-xs transition">
                                <i class="fa-solid fa-trash"></i>
                            </button>
                        </div>
                    </td>
                </tr>
            `).join('');
        }

        function openAddMemberModal() {
            document.getElementById('modalMemberTitle').textContent = "Tambah Anggota Baru";
            document.getElementById('formMemberOriginalId').value = "";
            document.getElementById('formMemberId').value = `SPW-10005-00${members.length + 1}`;
            document.getElementById('formMemberName').value = "";
            document.getElementById('formMemberKelas').value = "";
            document.getElementById('modalMemberForm').classList.remove('hidden');
        }

        function editMember(id) {
            const member = members.find(m => m.id === id);
            if (!member) return;

            document.getElementById('modalMemberTitle').textContent = "Edit Data Anggota";
            document.getElementById('formMemberOriginalId').value = member.id;
            document.getElementById('formMemberId').value = member.id;
            document.getElementById('formMemberName').value = member.name;
            document.getElementById('formMemberKelas').value = member.kelas;
            document.getElementById('formMemberTingkat').value = member.tingkat;
            document.getElementById('formMemberGender').value = member.gender;

            document.getElementById('modalMemberForm').classList.remove('hidden');
        }

        function closeMemberModal() {
            document.getElementById('modalMemberForm').classList.add('hidden');
        }

        function saveMember(e) {
            e.preventDefault();
            const origId = document.getElementById('formMemberOriginalId').value;
            const newId = document.getElementById('formMemberId').value.trim();
            const name = document.getElementById('formMemberName').value.trim();
            const kelas = document.getElementById('formMemberKelas').value.trim();
            const tingkat = document.getElementById('formMemberTingkat').value;
            const gender = document.getElementById('formMemberGender').value;

            if (origId) {
                const idx = members.findIndex(m => m.id === origId);
                if (idx !== -1) {
                    members[idx] = { id: newId, name, kelas, tingkat, gender };
                }
            } else {
                if (members.some(m => m.id === newId)) {
                    showToast("ID sudah terdaftar!", "error");
                    return;
                }
                members.push({ id: newId, name, kelas, tingkat, gender });
            }

            saveData();
            renderMembersTable();
            updateStats();
            closeMemberModal();
            showToast("Data anggota berhasil disimpan", "success");
        }

        function deleteMember(id) {
            if (confirm(`Apakah Anda yakin ingin menghapus anggota dengan ID ${id}?`)) {
                members = members.filter(m => m.id !== id);
                saveData();
                renderMembersTable();
                updateStats();
                showToast("Anggota berhasil dihapus", "success");
            }
        }

        // QR CARD GENERATOR MODAL
        function openQrModal(id) {
            const member = members.find(m => m.id === id);
            if (!member) return;

            document.getElementById('cardMemberName').textContent = member.name;
            document.getElementById('cardMemberId').textContent = member.id;
            document.getElementById('cardMemberKelas').textContent = `Kelas: ${member.kelas}`;
            document.getElementById('cardMemberTingkat').textContent = member.tingkat;

            // Generate QR Code
            const qrContainer = document.getElementById('qrcodeDisplay');
            qrContainer.innerHTML = "";
            new QRCode(qrContainer, {
                text: member.id,
                width: 130,
                height: 130,
                colorDark: "#261204",
                colorLight: "#ffffff",
                correctLevel: QRCode.CorrectLevel.H
            });

            document.getElementById('modalQrCard').classList.remove('hidden');
        }

        function closeQrModal() {
            document.getElementById('modalQrCard').classList.add('hidden');
        }

        function printCardOnly() {
            window.print();
        }

        // =========================================================
        // REKAP & REPORT LOGIC
        // =========================================================
        function renderRekapTable() {
            const tbody = document.getElementById('rekapTableBody');
            const filterDate = document.getElementById('rekapFilterDate').value;
            const searchQuery = document.getElementById('rekapSearch').value.toLowerCase();
            const filterStatus = document.getElementById('rekapFilterStatus').value;

            document.getElementById('printMeta').textContent = `Tanggal Presensi: ${filterDate || 'Semua Tanggal'} | Status: ${filterStatus}`;

            let filtered = [...attendanceLogs];

            if (filterDate) {
                filtered = filtered.filter(l => l.date === filterDate);
            }
            if (searchQuery) {
                filtered = filtered.filter(l => l.memberName.toLowerCase().includes(searchQuery) || l.memberId.toLowerCase().includes(searchQuery));
            }
            if (filterStatus !== 'ALL') {
                filtered = filtered.filter(l => l.status === filterStatus);
            }

            if (filtered.length === 0) {
                tbody.innerHTML = `<tr><td colspan="7" class="text-center py-6 text-slate-400 text-xs">Tidak ada catatan presensi sesuai kriteria.</td></tr>`;
                return;
            }

            tbody.innerHTML = filtered.map(l => {
                let badgeClass = "bg-emerald-100 text-emerald-800";
                if (l.status === "Izin") badgeClass = "bg-blue-100 text-blue-800";
                if (l.status === "Sakit") badgeClass = "bg-amber-100 text-amber-800";
                if (l.status === "Alpha") badgeClass = "bg-rose-100 text-rose-800";

                return `
                    <tr class="hover:bg-slate-50 transition">
                        <td class="p-3 font-mono text-slate-500">${l.time} <span class="text-[10px] text-slate-400 block">${l.date}</span></td>
                        <td class="p-3 font-mono font-semibold text-scout-700">${l.memberId}</td>
                        <td class="p-3 font-bold text-slate-800">${l.memberName}</td>
                        <td class="p-3"><span class="bg-amber-100 text-amber-800 text-xs font-semibold px-2 py-0.5 rounded">${l.kelas}</span></td>
                        <td class="p-3 text-slate-600">${l.tingkat}</td>
                        <td class="p-3 text-center">
                            <span class="px-2.5 py-1 rounded-full text-xs font-bold ${badgeClass}">
                                ${l.status}
                            </span>
                        </td>
                        <td class="p-3 text-center no-print">
                            <button onclick="deleteLog(${l.id})" class="text-rose-500 hover:text-rose-700 text-xs p-1"><i class="fa-solid fa-trash"></i></button>
                        </td>
                    </tr>
                `;
            }).join('');
        }

        function deleteLog(id) {
            if (confirm("Hapus baris presensi ini?")) {
                attendanceLogs = attendanceLogs.filter(l => l.id !== id);
                saveData();
                renderRekapTable();
                updateStats();
                renderRecentScans();
                showToast("Presensi dihapus", "success");
            }
        }

        function exportToCSV() {
            if (attendanceLogs.length === 0) {
                showToast("Tidak ada data presensi untuk diekspor!", "warning");
                return;
            }

            let csv = "ID,Nama,Kelas/Jabatan,Tingkat,Tanggal,Waktu,Status\n";
            attendanceLogs.forEach(l => {
                csv += `"${l.memberId}","${l.memberName}","${l.kelas}","${l.tingkat}","${l.date}","${l.time}","${l.status}"\n`;
            });

            const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
            const link = document.createElement("a");
            const url = URL.createObjectURL(blob);
            link.setAttribute("href", url);
            link.setAttribute("download", `Rekap_Presensi_SPENWAT_SCOUT_${new Date().toISOString().split('T')[0]}.csv`);
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
        }

        function printRekapReport() {
            window.print();
        }

        // =========================================================
        // SETTINGS & DEMO MANAGEMENT
        // =========================================================
        function resetDemoData() {
            if (confirm("Ganti data saat ini dengan data sampel bawaan tanpa regu?")) {
                members = [...DEFAULT_MEMBERS];
                attendanceLogs = [];
                saveData();
                updateStats();
                renderRecentScans();
                showToast("Data sampel berhasil dimuat ulang", "success");
            }
        }

        function clearAttendanceLogs() {
            if (confirm("Apakah Anda yakin ingin MENGHAPUS SEMUA LOG PRESENSI? Action ini tidak dapat dibatalkan.")) {
                attendanceLogs = [];
                saveData();
                updateStats();
                renderRecentScans();
                if (!document.getElementById('sec-rekap').classList.contains('hidden')) {
                    renderRekapTable();
                }
                showToast("Seluruh riwayat presensi telah dibersihkan", "success");
            }
        }

        // UTILITY: TOAST NOTIFICATIONS
        function showToast(message, type = "success") {
            const container = document.getElementById('toastContainer');
            const toast = document.createElement('div');
            
            let bgClass = "bg-slate-900 text-white";
            let icon = "fa-circle-info";

            if (type === "success") {
                bgClass = "bg-emerald-700 text-white";
                icon = "fa-circle-check";
            } else if (type === "error") {
                bgClass = "bg-rose-700 text-white";
                icon = "fa-circle-xmark";
            } else if (type === "warning") {
                bgClass = "bg-amber-600 text-white";
                icon = "fa-triangle-exclamation";
            }

            toast.className = `px-4 py-3 rounded-xl shadow-xl flex items-center gap-3 text-xs font-semibold transform transition-all duration-300 translate-y-2 ${bgClass}`;
            toast.innerHTML = `<i class="fa-solid ${icon} text-base"></i> <span>${message}</span>`;

            container.appendChild(toast);

            setTimeout(() => {
                toast.classList.add('opacity-0', '-translate-y-2');
                setTimeout(() => toast.remove(), 300);
            }, 3000);
        }
    </script>
</body>
</html>
