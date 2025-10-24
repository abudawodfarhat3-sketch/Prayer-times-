v<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <meta name="author" content="Abu Dawod">
    <meta name="description" content="Islamic Prayer Times App for Saida, Lebanon - Made by Abu Dawod">
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="apple-mobile-web-app-title" content="أبو داود">
    <meta name="application-name" content="أبو داود">
    <meta name="theme-color" content="#16a34a">
    
    <!-- PWA Manifest -->
    <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoi2KPYqNmIINiv2KfZiNivIiwic2hvcnRfbmFtZSI6Itin2KjZiCDYr9in2YjYryIsInN0YXJ0X3VybCI6Ii4vaW5kZXguaHRtbCIsImRpc3BsYXkiOiJzdGFuZGFsb25lIiwiYmFja2dyb3VuZF9jb2xvciI6IiNmZmZmZmYiLCJ0aGVtZV9jb2xvciI6IiMxNmEzNGEiLCJpY29ucyI6W3sic3JjIjoiZGF0YTppbWFnZS9zdmcreG1sO2Jhc2U2NCxQSE4yWnlCNGJXeHVjejBpYUhSMGNEb3ZMM2QzZHk1M015NXZjbWN2TWpBd01DOXpkbWNpSUhacFpYZENiM2c5SWpBZ01DQXlOQ0F5TkNJK1BHTnBjbU5zWlNCamVEMGlNVElpSUdONVBTSXhNaUlnY2owSU5pSWdabWxzYkQwaUl6RTJZVE0wWVNJdlBqd3ZjM1puUGc9PSIsInNpemVzIjoiNTEyeDUxMiIsInR5cGUiOiJpbWFnZS9zdmcreG1sIn1dfQ==">
    
    <!-- Favicon / App Icon -->
    <link rel="icon" type="image/svg+xml" href="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI+PGNpcmNsZSBjeD0iMTIiIGN5PSIxMiIgcj0iNiIgZmlsbD0iIzE2YTM0YSIvPjwvc3ZnPg==">
    
    <title>أبو داود - Prayer Times</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .animate-pulse { animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite; }
        @keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: .5; } }
        .animate-spin { animation: spin 1s linear infinite; }
        @keyframes spin { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }
        .transition-all { transition: all 0.3s ease; }
        .transform:hover { transform: scale(1.02); }
        .tab-button { transition: all 0.3s ease; }
        .tab-button:hover { background-color: rgba(0,0,0,0.05); }
    </style>
</head>
<body class="bg-gradient-to-br from-green-50 to-blue-50 min-h-screen">
    <div id="app"></div>

    <script>
        const app = {
            state: {
                currentTime: new Date(),
                activeTab: 'prayer',
                notificationsEnabled: localStorage.getItem('notificationsEnabled') === 'true',
                selectedAthkar: 'morning',
                selectedMuadhun: 'local1',
                isPlayingAdhan: false,
                currentAudio: null,
                uploadedAudio: null,
                prayerTimes: {
                    fajr: '05:30',
                    sunrise: '06:45',
                    dhuhr: '12:15',
                    asr: '15:45',
                    maghrib: '18:20',
                    isha: '19:45'
                },
                isLoading: false,
                lastUpdated: null,
                connectionStatus: 'loading',
                currentHijriDate: '',
                customLogo: localStorage.getItem('customLogo') || null
            },
            
            SAIDA: { latitude: 33.5631, longitude: 35.3708, city: 'Saida', country: 'Lebanon' },
            
            muadhuns: {
                local1: { name: 'Your Adhan 1 (haii3alslah936584)', desc: 'Local MP3 file from Downloads', url: './haii3alslah936584 (1).mp3' },
                local2: { name: 'Your Adhan 2 (haii3alslah2336584)', desc: 'Local MP3 file from Downloads', url: './haii3alslah2336584.mp3' },
                upload: { name: 'Upload Your Own', desc: 'Click to upload any MP3 file', url: null }
            },
            
            athkar: {
                morning: [
                    { ar: 'اللَّهُمَّ أَعِنِّي عَلَى ذِكْرِكَ وَشُكْرِكَ وَحُسْنِ عِبَادَتِكَ', tr: 'Allahumma a\'inni ala dhikrika wa shukrika wa husni ibadatika', en: 'O Allah, help me to remember You, to thank You, and to worship You in the best manner.' },
                    { ar: 'أَصْبَحْنَا وَأَصْبَحَ الْمُلْكُ لِلَّهِ', tr: 'Asbahna wa asbaha al-mulku lillahi', en: 'We have entered the morning and the whole kingdom belongs to Allah.' },
                    { ar: 'اللَّهُمَّ بِكَ أَصْبَحْنَا وَبِكَ أَمْسَيْنَا وَبِكَ نَحْيَا وَبِكَ نَمُوتُ وَإِلَيْكَ النُّشُورُ', tr: 'Allahumma bika asbahna wa bika amsayna wa bika nahya wa bika namutu wa ilayka an-nushur', en: 'O Allah, by You we enter the morning and by You we enter the evening, by You we live and by You we die, and to You is the resurrection.' }
                ],
                evening: [
                    { ar: 'أَمْسَيْنَا وَأَمْسَى الْمُلْكُ لِلَّهِ', tr: 'Amsayna wa amsa al-mulku lillahi', en: 'We have entered the evening and the whole kingdom belongs to Allah.' },
                    { ar: 'اللَّهُمَّ إِنِّي أَعُوذُ بِكَ مِنْ عَذَابِ جَهَنَّمَ وَمِنْ عَذَابِ الْقَبْرِ', tr: 'Allahumma inni a\'udhu bika min adhabi jahannama wa min adhabi al-qabr', en: 'O Allah, I seek refuge in You from the punishment of Hell and from the punishment of the grave.' }
                ],
                general: [
                    { ar: 'سُبْحَانَ اللَّهِ وَبِحَمْدِهِ', tr: 'Subhan Allahi wa bihamdihi', en: 'Glory be to Allah and praise be to Him.' },
                    { ar: 'لَا إِلَهَ إِلَّا اللَّهُ وَحْدَهُ لَا شَرِيكَ لَهُ', tr: 'La ilaha illa Allah wahdahu la sharika lahu', en: 'There is no god but Allah alone, with no partner.' },
                    { ar: 'أَسْتَغْفِرُ اللَّهَ الْعَظِيمَ', tr: 'Astaghfiru Allah al-adheem', en: 'I seek forgiveness from Allah, the Most Great.' }
                ]
            },
            
            tawbaReminders: [
                'Remember to seek Allah\'s forgiveness regularly - He is Most Forgiving, Most Merciful.',
                'Turn to Allah in repentance before it\'s too late. Every moment is a chance for a new beginning.',
                'Allah loves those who turn to Him in repentance and purify themselves.',
                'No sin is too great for Allah\'s forgiveness when you sincerely repent.',
                'Make Istighfar (seeking forgiveness) a daily habit - it brings peace to the heart.'
            ],
            
            async fetchPrayerTimes() {
                this.state.isLoading = true;
                this.state.connectionStatus = 'connecting';
                this.render();
                
                try {
                    const d = new Date();
                    const url = `https://api.aladhan.com/v1/timings/${d.getDate()}-${d.getMonth()+1}-${d.getFullYear()}?latitude=${this.SAIDA.latitude}&longitude=${this.SAIDA.longitude}&method=8`;
                    const res = await fetch(url);
                    const data = await res.json();
                    
                    // Convert 24hr to 12hr format
                    const convert24to12 = (time24) => {
                        const [hours, minutes] = time24.split(':');
                        const h = parseInt(hours);
                        const period = h >= 12 ? 'PM' : 'AM';
                        const h12 = h === 0 ? 12 : h > 12 ? h - 12 : h;
                        return `${h12}:${minutes} ${period}`;
                    };
                    
                    this.state.prayerTimes = {
                        fajr: convert24to12(data.data.timings.Fajr),
                        sunrise: convert24to12(data.data.timings.Sunrise),
                        dhuhr: convert24to12(data.data.timings.Dhuhr),
                        asr: convert24to12(data.data.timings.Asr),
                        maghrib: convert24to12(data.data.timings.Maghrib),
                        isha: convert24to12(data.data.timings.Isha)
                    };
                    
                    this.state.currentHijriDate = `${data.data.date.hijri.day} ${data.data.date.hijri.month.en} ${data.data.date.hijri.year}`;
                    this.state.connectionStatus = 'connected';
                    this.state.lastUpdated = new Date();
                } catch(e) {
                    console.error(e);
                    this.state.connectionStatus = 'offline';
                }
                
                this.state.isLoading = false;
                this.render();
            },
            
            getNextPrayer() {
                const now = new Date();
                const currentHour = now.getHours();
                const currentMin = now.getMinutes();
                const currentTimeInMin = currentHour * 60 + currentMin;
                
                // Convert 12hr format back to minutes for comparison
                const timeToMinutes = (time12) => {
                    const match = time12.match(/(\d+):(\d+)\s*(AM|PM)/i);
                    if (!match) return 0;
                    let hours = parseInt(match[1]);
                    const minutes = parseInt(match[2]);
                    const period = match[3].toUpperCase();
                    
                    if (period === 'PM' && hours !== 12) hours += 12;
                    if (period === 'AM' && hours === 12) hours = 0;
                    
                    return hours * 60 + minutes;
                };
                
                const prayers = [
                    ['fajr', this.state.prayerTimes.fajr],
                    ['dhuhr', this.state.prayerTimes.dhuhr],
                    ['asr', this.state.prayerTimes.asr],
                    ['maghrib', this.state.prayerTimes.maghrib],
                    ['isha', this.state.prayerTimes.isha]
                ];
                
                for(let [prayer, time] of prayers) {
                    const prayerTimeInMin = timeToMinutes(time);
                    if(prayerTimeInMin > currentTimeInMin) {
                        // Calculate time remaining
                        const diffMin = prayerTimeInMin - currentTimeInMin;
                        const hours = Math.floor(diffMin / 60);
                        const minutes = diffMin % 60;
                        
                        let remaining = '';
                        if (hours > 0) {
                            remaining = `${hours} ساعة و ${minutes} دقيقة`;
                        } else {
                            remaining = `${minutes} دقيقة`;
                        }
                        
                        this.state.timeRemaining = remaining;
                        return {name: prayer, time: time};
                    }
                }
                
                // Next day's Fajr
                const fajrTimeInMin = timeToMinutes(this.state.prayerTimes.fajr);
                const diffMin = (24 * 60) - currentTimeInMin + fajrTimeInMin;
                const hours = Math.floor(diffMin / 60);
                const minutes = diffMin % 60;
                this.state.timeRemaining = `${hours} ساعة و ${minutes} دقيقة`;
                
                return {name: 'fajr', time: this.state.prayerTimes.fajr};
            },
            
            
            
            handleLogoUpload(event) {
                const file = event.target.files[0];
                if (file && file.type.startsWith('image/')) {
                    const reader = new FileReader();
                    reader.onload = (e) => {
                        this.state.customLogo = e.target.result;
                        localStorage.setItem('customLogo', e.target.result);
                        alert('✅ Logo uploaded successfully!');
                        this.render();
                    };
                    reader.readAsDataURL(file);
                } else {
                    alert('Please upload a valid image file (PNG, JPG, etc.)');
                }
            },
            
            shareApp() {
                const shareData = {
                    title: 'أبو داود - Islamic Prayer App',
                    text: 'Check out this beautiful Islamic prayer times app for Saida, Lebanon! Features: Daily prayer times, Adhan audio, Athkar, and more. Made by Abu Dawod.',
                    url: window.location.href
                };
                
                if (navigator.share) {
                    navigator.share(shareData)
                        .then(() => {
                            document.getElementById('shareStatus').textContent = '✅ Shared successfully!';
                            setTimeout(() => {
                                document.getElementById('shareStatus').textContent = '';
                            }, 3000);
                        })
                        .catch((error) => {
                            if (error.name !== 'AbortError') {
                                this.copyLink();
                            }
                        });
                } else {
                    this.copyLink();
                }
            },
            
            copyLink() {
                const link = window.location.href;
                
                if (navigator.clipboard && navigator.clipboard.writeText) {
                    navigator.clipboard.writeText(link)
                        .then(() => {
                            document.getElementById('shareStatus').textContent = '✅ Link copied to clipboard!';
                            setTimeout(() => {
                                document.getElementById('shareStatus').textContent = '';
                            }, 3000);
                        })
                        .catch(() => {
                            this.fallbackCopyLink(link);
                        });
                } else {
                    this.fallbackCopyLink(link);
                }
            },
            
            fallbackCopyLink(link) {
                const textArea = document.createElement('textarea');
                textArea.value = link;
                textArea.style.position = 'fixed';
                textArea.style.left = '-999999px';
                document.body.appendChild(textArea);
                textArea.focus();
                textArea.select();
                
                try {
                    document.execCommand('copy');
                    document.getElementById('shareStatus').textContent = '✅ Link copied! Share it with WhatsApp, Telegram, etc.';
                    setTimeout(() => {
                        document.getElementById('shareStatus').textContent = '';
                    }, 5000);
                } catch (err) {
                    document.getElementById('shareStatus').textContent = '⚠️ Please manually copy: ' + link;
                }
                
                document.body.removeChild(textArea);
            },
            
            handleAudioUpload(event) {
                const file = event.target.files[0];
                if (file && file.type.startsWith('audio/')) {
                    const audioUrl = URL.createObjectURL(file);
                    this.state.uploadedAudio = audioUrl;
                    alert('✅ Audio file uploaded successfully! Click Play to listen.');
                    this.render();
                } else {
                    alert('Please upload a valid audio file (MP3, WAV, etc.)');
                }
            },
            
            playAdhan() {
                if(this.state.isPlayingAdhan && this.state.currentAudio) {
                    this.state.currentAudio.pause();
                    this.state.currentAudio.currentTime = 0;
                    this.state.isPlayingAdhan = false;
                    this.state.currentAudio = null;
                    this.render();
                    return;
                }
                
                // Use uploaded audio, or selected muadhun URL
                let audioSource = this.state.uploadedAudio || this.muadhuns[this.state.selectedMuadhun].url;
                
                if (!audioSource) {
                    alert('Please upload an MP3 file first, or make sure the MP3 files are in the same folder as this HTML file.');
                    return;
                }
                
                const audio = new Audio(audioSource);
                
                audio.addEventListener('error', (e) => {
                    console.error('Audio error:', e);
                    alert('⚠️ Cannot load audio file.\n\n✅ INSTRUCTIONS:\n1. Save this HTML file in your Downloads folder\n2. Make sure your MP3 files are in the same folder\n3. Or use the "Upload Audio" button to select any MP3 file');
                    this.state.isPlayingAdhan = false;
                    this.render();
                });
                
                audio.play().then(() => {
                    console.log('✅ Audio playing!');
                    this.state.isPlayingAdhan = true;
                    this.state.currentAudio = audio;
                    this.render();
                }).catch((err) => {
                    console.error('Play error:', err);
                    if (err.name === 'NotAllowedError') {
                        alert('Browser blocked audio! Click anywhere on the page first, then try again.');
                    } else {
                        alert('⚠️ Cannot play audio.\n\n✅ SOLUTION: Use the "Upload Audio" button to select your MP3 file manually.');
                    }
                    this.state.isPlayingAdhan = false;
                    this.render();
                });
                
                audio.onended = () => {
                    this.state.isPlayingAdhan = false;
                    this.state.currentAudio = null;
                    this.render();
                };
            },
            
            async toggleNotifications() {
                if(!this.state.notificationsEnabled) {
                    if('Notification' in window) {
                        const perm = await Notification.requestPermission();
                        if(perm === 'granted') {
                            this.state.notificationsEnabled = true;
                            localStorage.setItem('notificationsEnabled', 'true');
                        }
                    }
                } else {
                    this.state.notificationsEnabled = false;
                    localStorage.setItem('notificationsEnabled', 'false');
                }
                this.render();
            },
            
            icon(name) {
                const icons = {
                    moon: '<path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path>',
                    bell: '<path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"></path><path d="M13.73 21a2 2 0 0 1-3.46 0"></path>',
                    clock: '<circle cx="12" cy="12" r="10"></circle><polyline points="12 6 12 12 16 14"></polyline>',
                    sun: '<circle cx="12" cy="12" r="5"></circle><line x1="12" y1="1" x2="12" y2="3"></line><line x1="12" y1="21" x2="12" y2="23"></line><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line><line x1="1" y1="12" x2="3" y2="12"></line><line x1="21" y1="12" x2="23" y2="12"></line><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line>',
                    book: '<path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20"></path><path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z"></path>',
                    heart: '<path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"></path>',
                    wifi: '<path d="M5 12.55a11 11 0 0 1 14.08 0"></path><path d="M1.42 9a16 16 0 0 1 21.16 0"></path><path d="M8.53 16.11a6 6 0 0 1 6.95 0"></path><line x1="12" y1="20" x2="12.01" y2="20"></line>',
                    wifioff: '<line x1="1" y1="1" x2="23" y2="23"></line><path d="M16.72 11.06A10.94 10.94 0 0 1 19 12.55"></path><path d="M5 12.55a10.94 10.94 0 0 1 5.17-2.39"></path><path d="M10.71 5.05A16 16 0 0 1 22.58 9"></path><path d="M1.42 9a15.91 15.91 0 0 1 4.7-2.88"></path><path d="M8.53 16.11a6 6 0 0 1 6.95 0"></path><line x1="12" y1="20" x2="12.01" y2="20"></line>',
                    refresh: '<polyline points="23 4 23 10 17 10"></polyline><polyline points="1 20 1 14 7 14"></polyline><path d="M3.51 9a9 9 0 0 1 14.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0 0 20.49 15"></path>',
                    pin: '<path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z"></path><circle cx="12" cy="10" r="3"></circle>'
                };
                return `<svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">${icons[name]}</svg>`;
            },
            
            
            translatePrayerName(prayer) {
                const names = {
                    fajr: 'الفجر',
                    sunrise: 'الشروق',
                    dhuhr: 'الظهر',
                    asr: 'العصر',
                    maghrib: 'المغرب',
                    isha: 'العشاء'
                };
                return names[prayer] || prayer;
            },
            
            render() {
                const s = this.state;
                const next = this.getNextPrayer();
                
                document.getElementById('app').innerHTML = `
                    <div class="min-h-screen">
                        <!-- App Header with Logo -->
                        <div class="bg-gradient-to-r from-green-600 to-blue-600 text-white p-6 shadow-lg">
                            <!-- App Logo -->
                            <div class="app-logo arabic-text">أد</div>
                            
                            <div class="flex items-center justify-between flex-wrap gap-4">
                                <div class="flex items-center space-x-3">
                                    <div>
                                        <h1 class="text-2xl font-bold arabic-text">أبو داود</h1>
                                        <p class="text-sm text-green-100">Prayer Times - Saida, Lebanon</p>
                                    </div>
                                </div>
                                <div class="text-right">
                                    <div class="text-2xl font-bold">${s.currentTime.toLocaleTimeString('en-US', {hour: 'numeric', minute: '2-digit', hour12: true})}</div>
                                    <div class="text-sm opacity-90">${s.currentTime.toLocaleDateString()}</div>
                                    ${s.currentHijriDate ? `<div class="text-xs opacity-75 mt-1 arabic-text">${s.currentHijriDate}</div>` : ''}
                                </div>
                            </div>
                            
                            <!-- Custom Logo Section -->
                            ${s.customLogo ? `
                                <div class="mt-6 text-center">
                                    <img src="${s.customLogo}" class="custom-logo-img mx-auto" style="max-height: 120px; border-radius: 15px; box-shadow: 0 5px 15px rgba(0,0,0,0.2);" alt="App Logo">
                                </div>
                            ` : ''}
                        </div>

                        <!-- Tabs -->
                        <div class="bg-white shadow-sm">
                            <div class="flex overflow-x-auto" dir="rtl">
                                ${[
                                    {id: 'prayer', label: 'الصلاة'},
                                    {id: 'adhan', label: 'الأذان'},
                                    {id: 'athkar', label: 'الأذكار'},
                                    {id: 'tawba', label: 'التوبة'},
                                    {id: 'profile', label: '👤 الملف'}
                                ].map(tab => `
                                    <button onclick="app.state.activeTab='${tab.id}'; app.render()" 
                                        class="tab-button flex-1 flex items-center justify-center space-x-2 py-4 px-6 border-b-2 transition-colors whitespace-nowrap ${
                                            s.activeTab === tab.id ? 'border-green-500 text-green-600 bg-green-50' : 'border-transparent text-gray-500 hover:text-gray-700'
                                        }">
                                        <span class="font-medium arabic-text">${tab.label}</span>
                                    </button>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Made by Abu Dawod Banner -->
                        <div class="p-6 pb-4">
                            <div class="bg-gradient-to-r from-green-600 to-blue-600 text-white p-4 rounded-xl shadow-lg text-center">
                                <p class="text-lg font-semibold arabic-text">صُنع بـ ❤️ من قبل أبو داود</p>
                                <p class="text-sm opacity-90">May Allah accept this work and make it beneficial for all Muslims</p>
                            </div>
                        </div>

                        <!-- Content -->
                        <div class="p-6 pt-0">
                            ${s.activeTab === 'prayer' ? this.renderPrayer(next) : ''}
                            ${s.activeTab === 'adhan' ? this.renderAdhan() : ''}
                            ${s.activeTab === 'athkar' ? this.renderAthkar() : ''}
                            ${s.activeTab === 'tawba' ? this.renderTawba() : ''}
                            ${s.activeTab === 'profile' ? this.renderProfile() : ''}
                        </div>
                    </div>
                `;
            },
            
            renderProfile() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <!-- Developer Profile Header -->
                        <div class="bg-gradient-to-r from-green-600 via-emerald-600 to-blue-600 text-white p-8 rounded-xl shadow-2xl">
                            <div class="text-center">
                                <!-- Profile Picture Upload Area -->
                                <div class="mb-6">
                                    <div class="w-32 h-32 mx-auto rounded-full bg-white bg-opacity-20 flex items-center justify-center cursor-pointer hover:bg-opacity-30 transition-all overflow-hidden border-4 border-white shadow-lg" onclick="document.getElementById('logoUpload').click()">
                                        ${s.customLogo ? `
                                            <img src="${s.customLogo}" class="w-full h-full object-cover" alt="Profile">
                                        ` : `
                                            <svg class="w-16 h-16 text-white opacity-75" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"></path>
                                            </svg>
                                        `}
                                    </div>
                                    <input 
                                        type="file" 
                                        id="logoUpload"
                                        accept="image/*" 
                                        onchange="app.handleLogoUpload(event)"
                                        style="display: none;"
                                    />
                                    <p class="text-sm mt-2 opacity-75">Click to upload profile photo</p>
                                </div>
                                
                                <h1 class="text-4xl font-bold mb-2 arabic-text">أبو داود</h1>
                                <p class="text-xl mb-2">Abu Dawod</p>
                                <p class="text-lg opacity-90">Islamic App Developer</p>
                                <div class="flex items-center justify-center gap-2 mt-4">
                                    <span class="px-4 py-1 bg-white bg-opacity-20 rounded-full text-sm">Saida, Lebanon 🇱🇧</span>
                                </div>
                            </div>
                        </div>

                        <!-- About This Project -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">📱 About This Application</h2>
                            <div class="space-y-4 text-gray-700">
                                <p class="leading-relaxed">
                                    <strong class="text-green-600">Islamic Prayer Times & Athkar App</strong> is a comprehensive web application designed to help Muslims stay connected with their daily prayers and remembrance of Allah.
                                </p>
                                <p class="leading-relaxed">
                                    This app provides accurate prayer times for Saida, Lebanon, beautiful Adhan audio, daily Athkar, and reminders for Tawba (repentance).
                                </p>
                            </div>
                        </div>

                        <!-- Features -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">✨ Key Features</h2>
                            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                                ${[
                                    { icon: '🕌', title: 'Daily Prayer Times', desc: 'Accurate times for Saida updated automatically' },
                                    { icon: '🔔', title: 'Prayer Notifications', desc: 'Get notified when it\'s time to pray' },
                                    { icon: '🎵', title: 'Beautiful Adhan', desc: 'Multiple Adhan styles with audio playback' },
                                    { icon: '📿', title: 'Daily Athkar', desc: 'Morning, evening, and general remembrances' },
                                    { icon: '💜', title: 'Tawba Reminders', desc: 'Encouragement for repentance and forgiveness' },
                                    { icon: '🌙', title: 'Hijri Calendar', desc: 'Display Islamic date alongside Gregorian' },
                                    { icon: '📱', title: 'Installable App', desc: 'Works like a native mobile application' },
                                    { icon: '🔄', title: 'Auto-Updates', desc: 'Prayer times update daily automatically' }
                                ].map(f => `
                                    <div class="flex items-start space-x-3 p-3 bg-green-50 rounded-lg">
                                        <span class="text-2xl">${f.icon}</span>
                                        <div>
                                            <h3 class="font-semibold text-gray-800">${f.title}</h3>
                                            <p class="text-sm text-gray-600">${f.desc}</p>
                                        </div>
                                    </div>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Technologies Used -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">🛠️ Technologies Used</h2>
                            <div class="flex flex-wrap gap-3">
                                ${['HTML5', 'CSS3', 'JavaScript', 'Tailwind CSS', 'AlAdhan API', 'Web Audio API', 'Local Storage', 'PWA'].map(tech => `
                                    <span class="px-4 py-2 bg-gradient-to-r from-green-100 to-blue-100 text-gray-800 rounded-full text-sm font-semibold border border-green-200">${tech}</span>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Statistics -->
                        <div class="bg-gradient-to-r from-purple-50 to-pink-50 p-6 rounded-xl shadow-lg border-2 border-purple-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">📊 App Statistics</h2>
                            <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
                                ${[
                                    { num: '5', label: 'Daily Prayers' },
                                    { num: '10+', label: 'Athkar Duas' },
                                    { num: '4', label: 'Adhan Styles' },
                                    { num: '100%', label: 'Free & Open' }
                                ].map(stat => `
                                    <div class="text-center p-4 bg-white rounded-lg shadow-sm">
                                        <div class="text-3xl font-bold text-green-600">${stat.num}</div>
                                        <div class="text-sm text-gray-600 mt-1">${stat.label}</div>
                                    </div>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Contact & Share -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">🤝 Share This App</h2>
                            <p class="text-gray-700 mb-4">Help spread the benefit! Share this Islamic prayer app with your community:</p>
                            <div class="space-y-3">
                                <button onclick="app.shareApp()" 
                                    class="w-full px-6 py-3 bg-gradient-to-r from-green-500 to-blue-500 text-white rounded-lg font-semibold hover:from-green-600 hover:to-blue-600 transition-all shadow-md">
                                    📤 Share App
                                </button>
                                <button onclick="app.copyLink()" 
                                    class="w-full px-6 py-3 bg-gray-100 text-gray-800 rounded-lg font-semibold hover:bg-gray-200 transition-all">
                                    📋 Copy App Link
                                </button>
                                <div class="text-center text-sm text-gray-500 mt-2">
                                    <p id="shareStatus"></p>
                                </div>
                            </div>
                        </div>

                        <!-- Credits & License -->
                        <div class="bg-gradient-to-r from-green-600 to-blue-600 text-white p-6 rounded-xl shadow-lg text-center">
                            <h2 class="text-2xl font-bold mb-3 arabic-text">بارك الله فيك</h2>
                            <p class="text-lg mb-4">May Allah bless this work and make it a source of continuous reward (Sadaqah Jariyah)</p>
                            <div class="text-sm opacity-90">
                                <p>Developed with ❤️ by <strong class="arabic-text">أبو داود</strong></p>
                                <p class="mt-2">Version 1.0.0 | 2025</p>
                                <p class="mt-2">Free for all Muslims to use and benefit from</p>
                            </div>
                        </div>

                        ${s.customLogo ? `
                            <div class="bg-white p-4 rounded-lg shadow-sm border border-gray-200 text-center">
                                <button 
                                    onclick="app.state.customLogo = null; localStorage.removeItem('customLogo'); app.render();"
                                    class="px-6 py-2 bg-red-50 hover:bg-red-100 text-red-600 rounded-lg font-medium transition-colors">
                                    🗑️ Remove Profile Photo
                                </button>
                            </div>
                        ` : ''}
                    </div>
                `;
            },
            
            renderSettings() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-purple-500 to-indigo-500 text-white p-6 rounded-xl shadow-lg">
                            <h2 class="text-2xl font-bold mb-2">⚙️ Settings</h2>
                            <p class="text-lg opacity-90">Upload your program image</p>
                        </div>

                        <!-- Upload Photo Only -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-lg font-semibold mb-4 text-center">📸 Program Image</h3>
                            
                            <div class="custom-logo-container" onclick="document.getElementById('logoUpload').click()">
                                ${s.customLogo ? `
                                    <img src="${s.customLogo}" class="custom-logo-img" alt="Program Image">
                                ` : `
                                    <div class="text-center text-gray-400">
                                        <svg class="w-16 h-16 mx-auto mb-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"></path>
                                        </svg>
                                        <p class="text-sm font-semibold">Click to upload photo</p>
                                    </div>
                                `}
                            </div>
                            
                            <input 
                                type="file" 
                                id="logoUpload"
                                accept="image/*" 
                                onchange="app.handleLogoUpload(event)"
                                style="display: none;"
                            />
                            
                            ${s.customLogo ? `
                                <button 
                                    onclick="app.state.customLogo = null; localStorage.removeItem('customLogo'); app.render();"
                                    class="w-full mt-4 px-4 py-3 bg-red-50 hover:bg-red-100 text-red-600 rounded-lg font-semibold transition-colors">
                                    🗑️ Remove Photo
                                </button>
                            ` : ''}
                        </div>
                    </div>
                `;
            },
            
            renderPrayer(next) {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-green-500 to-blue-500 text-white p-6 rounded-xl shadow-lg transform transition-all hover:shadow-2xl">
                            <div class="text-center">
                                <h3 class="text-lg font-semibold opacity-90 mb-2">الصلاة القادمة</h3>
                                <h2 class="text-4xl font-bold mb-2 arabic-text">${this.translatePrayerName(next.name)}</h2>
                                <p class="text-2xl opacity-90 mb-3">${next.time}</p>
                                
                                <!-- Time Remaining -->
                                <div class="bg-white bg-opacity-20 rounded-lg p-4 mt-4">
                                    <p class="text-sm opacity-90 mb-1">الوقت المتبقي</p>
                                    <p class="text-3xl font-bold arabic-text">${s.timeRemaining}</p>
                                </div>
                            </div>
                        </div>

                        <div class="bg-white p-4 rounded-lg shadow-sm border border-gray-200">
                            <div class="flex items-center justify-between mb-3">
                                <div class="flex items-center space-x-3 space-x-reverse">
                                    ${this.icon('bell')}
                                    <span class="font-medium arabic-text">تنبيهات الصلاة</span>
                                </div>
                                <button onclick="app.toggleNotifications()" 
                                    class="relative inline-flex h-6 w-11 items-center rounded-full transition-colors ${s.notificationsEnabled ? 'bg-green-600' : 'bg-gray-200'}">
                                    <span class="inline-block h-4 w-4 transform rounded-full bg-white transition-transform ${s.notificationsEnabled ? 'translate-x-6' : 'translate-x-1'}"></span>
                                </button>
                            </div>
                            <div class="flex items-center justify-between text-sm pt-3 border-t border-gray-100">
                                <div class="flex items-center space-x-2 space-x-reverse">
                                    ${s.connectionStatus === 'connected' ? `${this.icon('wifi')}<span class="text-green-600 arabic-text">متصل بصيدا</span>` : 
                                      s.connectionStatus === 'connecting' ? `<div class="animate-spin">${this.icon('refresh')}</div><span class="text-blue-600 arabic-text">جاري الاتصال...</span>` :
                                      `${this.icon('wifioff')}<span class="text-red-600 arabic-text">غير متصل</span>`}
                                </div>
                                ${s.lastUpdated ? `<span class="text-gray-500 arabic-text">آخر تحديث: ${s.lastUpdated.toLocaleTimeString('ar-LB', {hour: 'numeric', minute: '2-digit', hour12: true})}</span>` : ''}
                            </div>
                            <button onclick="app.fetchPrayerTimes()" ${s.isLoading ? 'disabled' : ''}
                                class="w-full mt-3 px-4 py-2 bg-blue-50 hover:bg-blue-100 text-blue-600 rounded-lg font-medium transition-colors disabled:opacity-50 arabic-text">
                                ${s.isLoading ? '🔄 جاري التحديث...' : '🔄 تحديث أوقات الصلاة'}
                            </button>
                        </div>

                        <div class="bg-white rounded-lg shadow-sm border border-gray-200 overflow-hidden">
                            <h3 class="text-lg font-semibold p-4 border-b border-gray-200 bg-gray-50 arabic-text text-right">أوقات الصلاة اليوم</h3>
                            <div class="divide-y divide-gray-100">
                                ${Object.entries(s.prayerTimes).map(([p,t]) => `
                                    <div class="flex items-center justify-between p-4 hover:bg-gray-50 transition-colors" dir="rtl">
                                        <div class="flex items-center space-x-3 space-x-reverse">
                                            <div class="w-3 h-3 rounded-full ${p === next.name ? 'bg-green-500 animate-pulse' : 'bg-gray-300'}"></div>
                                            <span class="font-medium text-gray-700 arabic-text text-lg">${this.translatePrayerName(p)}</span>
                                        </div>
                                        <span class="text-lg font-semibold text-gray-900">${t}</span>
                                    </div>
                                `).join('')}
                            </div>
                        </div>
                    </div>
                `;
            },
            
            renderAdhan() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-amber-500 to-orange-500 text-white p-6 rounded-xl shadow-lg">
                            <div class="flex items-center space-x-3 mb-4">
                                ${this.icon('sun')}
                                <h2 class="text-2xl font-bold">Adhan Player</h2>
                            </div>
                            <p class="text-lg opacity-90">Listen to the beautiful call to prayer</p>
                        </div>

                        <!-- Instructions -->
                        <div class="bg-blue-50 p-4 rounded-lg border-2 border-blue-200">
                            <h3 class="font-bold text-blue-800 mb-2">📋 Instructions:</h3>
                            <ol class="text-sm text-blue-700 space-y-1 list-decimal list-inside">
                                <li>Save this HTML file in your <strong>Downloads folder</strong></li>
                                <li>Your MP3 files should be in the <strong>same folder</strong></li>
                                <li>Or click "Choose File" below to upload any MP3</li>
                            </ol>
                        </div>

                        <!-- Upload Audio -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-lg font-semibold mb-4">📁 Upload Your Adhan MP3</h3>
                            <input
                                type="file"
                                accept="audio/*"
                                onchange="app.handleAudioUpload(event)"
                                class="block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-green-50 file:text-green-700 hover:file:bg-green-100 cursor-pointer"
                            />
                            ${s.uploadedAudio ? '<p class="text-green-600 text-sm mt-2">✅ Audio uploaded! Ready to play.</p>' : ''}
                        </div>

                        <!-- Choose Adhan -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-lg font-semibold mb-4">Choose Adhan Source</h3>
                            <div class="space-y-3">
                                ${Object.entries(this.muadhuns).map(([k,m]) => `
                                    <div onclick="app.state.selectedMuadhun='${k}'; app.render()" 
                                        class="p-4 rounded-lg border-2 cursor-pointer transition-all transform hover:scale-102 ${
                                            s.selectedMuadhun === k ? 'border-amber-500 bg-amber-50 shadow-md' : 'border-gray-200 hover:border-gray-300'
                                        }">
                                        <div class="flex items-center justify-between">
                                            <div>
                                                <h4 class="font-semibold text-gray-800">${m.name}</h4>
                                                <p class="text-sm text-gray-600">${m.desc}</p>
                                            </div>
                                            <div class="w-4 h-4 rounded-full border-2 ${s.selectedMuadhun === k ? 'border-amber-500 bg-amber-500' : 'border-gray-300'}"></div>
                                        </div>
                                    </div>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Play Button -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <div class="text-center space-y-4">
                                <h3 class="text-lg font-semibold">${this.muadhuns[s.selectedMuadhun].name}</h3>
                                <button onclick="app.playAdhan()" 
                                    class="px-8 py-4 rounded-full text-white font-semibold text-lg transition-all transform hover:scale-105 shadow-lg ${
                                        s.isPlayingAdhan ? 'bg-red-500 hover:bg-red-600 animate-pulse' : 'bg-gradient-to-r from-green-500 to-blue-500 hover:from-green-600 hover:to-blue-600'
                                    }">
                                    ${s.isPlayingAdhan ? '⏸️ Stop Adhan' : '▶️ Play Adhan'}
                                </button>
                                ${s.isPlayingAdhan ? `
                                    <div class="bg-green-50 p-4 rounded-lg border border-green-200 animate-pulse">
                                        <p class="text-green-700 font-medium">🎵 Playing Adhan...</p>
                                        <p class="text-sm text-gray-600 mt-1">${s.uploadedAudio ? 'Your uploaded audio' : this.muadhuns[s.selectedMuadhun].name}</p>
                                    </div>
                                ` : ''}
                                <p class="text-sm text-gray-500 mt-2">🔊 Make sure your volume is on</p>
                            </div>
                        </div>
                    </div>
                `;
            },
            
            renderAthkar() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-white p-4 rounded-lg shadow-sm border border-gray-200">
                            <div class="flex space-x-2 overflow-x-auto">
                                ${Object.keys(this.athkar).map(cat => `
                                    <button onclick="app.state.selectedAthkar='${cat}'; app.render()" 
                                        class="px-4 py-2 rounded-lg font-medium capitalize transition-all whitespace-nowrap ${
                                            s.selectedAthkar === cat ? 'bg-green-500 text-white shadow-md' : 'bg-gray-100 text-gray-700 hover:bg-gray-200'
                                        }">
                                        ${cat}
                                    </button>
                                `).join('')}
                            </div>
                        </div>

                        ${this.athkar[s.selectedAthkar].map(d => `
                            <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200 transform transition-all hover:shadow-lg">
                                <div class="text-right mb-4">
                                    <p class="text-2xl text-green-700 leading-relaxed">${d.ar}</p>
                                </div>
                                <div class="space-y-2">
                                    <p class="text-gray-600 italic text-sm">${d.tr}</p>
                                    <p class="text-gray-800 font-medium">${d.en}</p>
                                </div>
                            </div>
                        `).join('')}
                    </div>
                `;
            },
            
            renderTawba() {
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-purple-500 to-pink-500 text-white p-6 rounded-xl shadow-lg">
                            <div class="flex items-center space-x-3 space-x-reverse mb-4" dir="rtl">
                                ${this.icon('heart')}
                                <h2 class="text-2xl font-bold arabic-text">التوبة - الاستغفار</h2>
                            </div>
                            <p class="text-lg opacity-90 arabic-text text-right leading-relaxed">
                                "وَمَن تَابَ بَعْدَ ظُلْمِهِ وَأَصْلَحَ فَإِنَّ اللَّهَ يَتُوبُ عَلَيْهِ ۗ إِنَّ اللَّهَ غَفُورٌ رَّحِيمٌ" - سورة المائدة 39
                            </p>
                        </div>

                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-xl font-semibold mb-4 text-center arabic-text">الاستغفار اليومي</h3>
                            <div class="text-center">
                                <div class="text-4xl font-bold text-green-600 mb-4 arabic-text leading-relaxed">
                                    أَسْتَغْفِرُ اللَّهَ الْعَظِيمَ
                                </div>
                                <p class="text-gray-600 mb-2 italic">Astaghfiru Allah al-Adheem</p>
                                <p class="text-sm text-gray-500 mb-4 arabic-text">أستغفر الله العظيم وأتوب إليه</p>
                            </div>
                        </div>

                        <div class="space-y-4">
                            <h3 class="text-xl font-semibold text-gray-700 arabic-text text-right">أدعية التوبة والاستغفار</h3>
                            
                            <div class="bg-gradient-to-r from-blue-50 to-purple-50 p-6 rounded-lg border-l-4 border-purple-400">
                                <p class="text-2xl text-gray-800 arabic-text text-right leading-loose mb-3">
                                    اللَّهُمَّ إِنَّكَ عَفُوٌّ كَرِيمٌ تُحِبُّ الْعَفْوَ فَاعْفُ عَنَّا
                                </p>
                                <p class="text-sm text-gray-600 italic">Allahumma innaka 'afuwwun karimun tuhibbul-'afwa fa'fu 'anna</p>
                                <p class="text-sm text-gray-700 mt-2 arabic-text text-right">
                                    اللهم إنك عفو كريم تحب العفو فاعف عنا
                                </p>
                            </div>

                            <div class="bg-gradient-to-r from-green-50 to-blue-50 p-6 rounded-lg border-l-4 border-green-400">
                                <p class="text-2xl text-gray-800 arabic-text text-right leading-loose mb-3">
                                    رَبَّنَا آتِنَا فِي الدُّنْيَا حَسَنَةً وَفِي الْآخِرَةِ حَسَنَةً وَقِنَا عَذَابَ النَّارِ
                                </p>
                                <p class="text-sm text-gray-600 italic">Rabbana atina fid-dunya hasanatan wa fil-akhirati hasanatan wa qina 'adhaban-nar</p>
                                <p class="text-sm text-gray-700 mt-2 arabic-text text-right">
                                    اللهم آتنا في الدنيا حسنة وفي الآخرة حسنة وقنا عذاب النار
                                </p>
                            </div>

                            <div class="bg-gradient-to-r from-purple-50 to-pink-50 p-6 rounded-lg border-l-4 border-pink-400">
                                <p class="text-2xl text-gray-800 arabic-text text-right leading-loose mb-3">
                                    اللَّهُمَّ آتِ نَفْسِي تَقْوَاهَا وَزَكِّهَا أَنتَ خَيْرُ مَن زَكَّاهَا أَنتَ وَلِيُّهَا وَمَوْلَاهَا
                                </p>
                                <p class="text-sm text-gray-600 italic">Allahumma ati nafsi taqwaha wa zakkiha anta khayru man zakkaha anta waliyyuha wa mawlaha</p>
                                <p class="text-sm text-gray-700 mt-2 arabic-text text-right">
                                    اللهم آت نفسي تقواها وزكها أنت خير من زكاها أنت وليها ومولاها
                                </p>
                            </div>
                        </div>

                        <div class="space-y-4">
                            <h3 class="text-xl font-semibold text-gray-700 arabic-text text-right">تذكيرات يومية</h3>
                            ${this.tawbaReminders.map(r => `
                                <div class="bg-gradient-to-r from-blue-50 to-purple-50 p-4 rounded-lg border-r-4 border-purple-400 transform transition-all hover:shadow-md" dir="rtl">
                                    <p class="text-gray-700 arabic-text text-right">${r}</p>
                                </div>
                            `).join('')}
                        </div>

                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-xl font-semibold mb-4 arabic-text text-right">دعاء الاستغفار</h3>
                            <div class="border-r-4 border-green-400 pr-4" dir="rtl">
                                <p class="text-2xl text-green-700 mb-3 text-right leading-loose arabic-text">
                                    رَبَّنَا ظَلَمْنَا أَنفُسَنَا وَإِن لَّمْ تَغْفِرْ لَنَا وَتَرْحَمْنَا لَنَكُونَنَّ مِنَ الْخَاسِرِينَ
                                </p>
                                <p class="text-sm text-gray-600 italic mb-1">
                                    Rabbana zalamna anfusana wa in lam taghfir lana wa tarhamna lanakoonanna minal-khasireen
                                </p>
                                <p class="text-sm text-gray-700 arabic-text text-right">
                                    "رَبَّنَا ظَلَمْنَا أَنفُسَنَا وَإِن لَّمْ تَغْفِرْ لَنَا وَتَرْحَمْنَا لَنَكُونَنَّ مِنَ الْخَاسِرِينَ"
                                </p>
                            </div>
                        </div>
                    </div>
                `;
            },
            
            init() {
                this.fetchPrayerTimes();
                setInterval(() => {
                    this.state.currentTime = new Date();
                    this.render();
                }, 1000);
                setInterval(() => this.fetchPrayerTimes(), 21600000);
                setInterval(() => {
                    const now = new Date();
                    if(now.getHours() === 0 && now.getMinutes() === 0) {
                        this.fetchPrayerTimes();
                    }
                }, 60000);
                this.render();
            }
        };
        
        window.app = app;
        app.init();
    </script>
</body>
</html><!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <meta name="author" content="Abu Dawod">
    <meta name="description" content="Islamic Prayer Times App for Saida, Lebanon - Made by Abu Dawod">
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="apple-mobile-web-app-title" content="أبو داود">
    <meta name="application-name" content="أبو داود">
    <meta name="theme-color" content="#16a34a">
    
    <!-- PWA Manifest -->
    <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoi2KPYqNmIINiv2KfZiNivIiwic2hvcnRfbmFtZSI6Itin2KjZiCDYr9in2YjYryIsInN0YXJ0X3VybCI6Ii4vaW5kZXguaHRtbCIsImRpc3BsYXkiOiJzdGFuZGFsb25lIiwiYmFja2dyb3VuZF9jb2xvciI6IiNmZmZmZmYiLCJ0aGVtZV9jb2xvciI6IiMxNmEzNGEiLCJpY29ucyI6W3sic3JjIjoiZGF0YTppbWFnZS9zdmcreG1sO2Jhc2U2NCxQSE4yWnlCNGJXeHVjejBpYUhSMGNEb3ZMM2QzZHk1M015NXZjbWN2TWpBd01DOXpkbWNpSUhacFpYZENiM2c5SWpBZ01DQXlOQ0F5TkNJK1BHTnBjbU5zWlNCamVEMGlNVElpSUdONVBTSXhNaUlnY2owSU5pSWdabWxzYkQwaUl6RTJZVE0wWVNJdlBqd3ZjM1puUGc9PSIsInNpemVzIjoiNTEyeDUxMiIsInR5cGUiOiJpbWFnZS9zdmcreG1sIn1dfQ==">
    
    <!-- Favicon / App Icon -->
    <link rel="icon" type="image/svg+xml" href="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI+PGNpcmNsZSBjeD0iMTIiIGN5PSIxMiIgcj0iNiIgZmlsbD0iIzE2YTM0YSIvPjwvc3ZnPg==">
    
    <title>أبو داود - Prayer Times</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .animate-pulse { animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite; }
        @keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: .5; } }
        .animate-spin { animation: spin 1s linear infinite; }
        @keyframes spin { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }
        .transition-all { transition: all 0.3s ease; }
        .transform:hover { transform: scale(1.02); }
        .tab-button { transition: all 0.3s ease; }
        .tab-button:hover { background-color: rgba(0,0,0,0.05); }
    </style>
</head>
<body class="bg-gradient-to-br from-green-50 to-blue-50 min-h-screen">
    <div id="app"></div>

    <script>
        const app = {
            state: {
                currentTime: new Date(),
                activeTab: 'prayer',
                notificationsEnabled: localStorage.getItem('notificationsEnabled') === 'true',
                selectedAthkar: 'morning',
                selectedMuadhun: 'local1',
                isPlayingAdhan: false,
                currentAudio: null,
                uploadedAudio: null,
                prayerTimes: {
                    fajr: '05:30',
                    sunrise: '06:45',
                    dhuhr: '12:15',
                    asr: '15:45',
                    maghrib: '18:20',
                    isha: '19:45'
                },
                isLoading: false,
                lastUpdated: null,
                connectionStatus: 'loading',
                currentHijriDate: '',
                customLogo: localStorage.getItem('customLogo') || null
            },
            
            SAIDA: { latitude: 33.5631, longitude: 35.3708, city: 'Saida', country: 'Lebanon' },
            
            muadhuns: {
                local1: { name: 'Your Adhan 1 (haii3alslah936584)', desc: 'Local MP3 file from Downloads', url: './haii3alslah936584 (1).mp3' },
                local2: { name: 'Your Adhan 2 (haii3alslah2336584)', desc: 'Local MP3 file from Downloads', url: './haii3alslah2336584.mp3' },
                upload: { name: 'Upload Your Own', desc: 'Click to upload any MP3 file', url: null }
            },
            
            athkar: {
                morning: [
                    { ar: 'اللَّهُمَّ أَعِنِّي عَلَى ذِكْرِكَ وَشُكْرِكَ وَحُسْنِ عِبَادَتِكَ', tr: 'Allahumma a\'inni ala dhikrika wa shukrika wa husni ibadatika', en: 'O Allah, help me to remember You, to thank You, and to worship You in the best manner.' },
                    { ar: 'أَصْبَحْنَا وَأَصْبَحَ الْمُلْكُ لِلَّهِ', tr: 'Asbahna wa asbaha al-mulku lillahi', en: 'We have entered the morning and the whole kingdom belongs to Allah.' },
                    { ar: 'اللَّهُمَّ بِكَ أَصْبَحْنَا وَبِكَ أَمْسَيْنَا وَبِكَ نَحْيَا وَبِكَ نَمُوتُ وَإِلَيْكَ النُّشُورُ', tr: 'Allahumma bika asbahna wa bika amsayna wa bika nahya wa bika namutu wa ilayka an-nushur', en: 'O Allah, by You we enter the morning and by You we enter the evening, by You we live and by You we die, and to You is the resurrection.' }
                ],
                evening: [
                    { ar: 'أَمْسَيْنَا وَأَمْسَى الْمُلْكُ لِلَّهِ', tr: 'Amsayna wa amsa al-mulku lillahi', en: 'We have entered the evening and the whole kingdom belongs to Allah.' },
                    { ar: 'اللَّهُمَّ إِنِّي أَعُوذُ بِكَ مِنْ عَذَابِ جَهَنَّمَ وَمِنْ عَذَابِ الْقَبْرِ', tr: 'Allahumma inni a\'udhu bika min adhabi jahannama wa min adhabi al-qabr', en: 'O Allah, I seek refuge in You from the punishment of Hell and from the punishment of the grave.' }
                ],
                general: [
                    { ar: 'سُبْحَانَ اللَّهِ وَبِحَمْدِهِ', tr: 'Subhan Allahi wa bihamdihi', en: 'Glory be to Allah and praise be to Him.' },
                    { ar: 'لَا إِلَهَ إِلَّا اللَّهُ وَحْدَهُ لَا شَرِيكَ لَهُ', tr: 'La ilaha illa Allah wahdahu la sharika lahu', en: 'There is no god but Allah alone, with no partner.' },
                    { ar: 'أَسْتَغْفِرُ اللَّهَ الْعَظِيمَ', tr: 'Astaghfiru Allah al-adheem', en: 'I seek forgiveness from Allah, the Most Great.' }
                ]
            },
            
            tawbaReminders: [
                'Remember to seek Allah\'s forgiveness regularly - He is Most Forgiving, Most Merciful.',
                'Turn to Allah in repentance before it\'s too late. Every moment is a chance for a new beginning.',
                'Allah loves those who turn to Him in repentance and purify themselves.',
                'No sin is too great for Allah\'s forgiveness when you sincerely repent.',
                'Make Istighfar (seeking forgiveness) a daily habit - it brings peace to the heart.'
            ],
            
            async fetchPrayerTimes() {
                this.state.isLoading = true;
                this.state.connectionStatus = 'connecting';
                this.render();
                
                try {
                    const d = new Date();
                    const url = `https://api.aladhan.com/v1/timings/${d.getDate()}-${d.getMonth()+1}-${d.getFullYear()}?latitude=${this.SAIDA.latitude}&longitude=${this.SAIDA.longitude}&method=8`;
                    const res = await fetch(url);
                    const data = await res.json();
                    
                    // Convert 24hr to 12hr format
                    const convert24to12 = (time24) => {
                        const [hours, minutes] = time24.split(':');
                        const h = parseInt(hours);
                        const period = h >= 12 ? 'PM' : 'AM';
                        const h12 = h === 0 ? 12 : h > 12 ? h - 12 : h;
                        return `${h12}:${minutes} ${period}`;
                    };
                    
                    this.state.prayerTimes = {
                        fajr: convert24to12(data.data.timings.Fajr),
                        sunrise: convert24to12(data.data.timings.Sunrise),
                        dhuhr: convert24to12(data.data.timings.Dhuhr),
                        asr: convert24to12(data.data.timings.Asr),
                        maghrib: convert24to12(data.data.timings.Maghrib),
                        isha: convert24to12(data.data.timings.Isha)
                    };
                    
                    this.state.currentHijriDate = `${data.data.date.hijri.day} ${data.data.date.hijri.month.en} ${data.data.date.hijri.year}`;
                    this.state.connectionStatus = 'connected';
                    this.state.lastUpdated = new Date();
                } catch(e) {
                    console.error(e);
                    this.state.connectionStatus = 'offline';
                }
                
                this.state.isLoading = false;
                this.render();
            },
            
            getNextPrayer() {
                const now = new Date();
                const currentHour = now.getHours();
                const currentMin = now.getMinutes();
                const currentTimeInMin = currentHour * 60 + currentMin;
                
                // Convert 12hr format back to minutes for comparison
                const timeToMinutes = (time12) => {
                    const match = time12.match(/(\d+):(\d+)\s*(AM|PM)/i);
                    if (!match) return 0;
                    let hours = parseInt(match[1]);
                    const minutes = parseInt(match[2]);
                    const period = match[3].toUpperCase();
                    
                    if (period === 'PM' && hours !== 12) hours += 12;
                    if (period === 'AM' && hours === 12) hours = 0;
                    
                    return hours * 60 + minutes;
                };
                
                const prayers = [
                    ['fajr', this.state.prayerTimes.fajr],
                    ['dhuhr', this.state.prayerTimes.dhuhr],
                    ['asr', this.state.prayerTimes.asr],
                    ['maghrib', this.state.prayerTimes.maghrib],
                    ['isha', this.state.prayerTimes.isha]
                ];
                
                for(let [prayer, time] of prayers) {
                    const prayerTimeInMin = timeToMinutes(time);
                    if(prayerTimeInMin > currentTimeInMin) {
                        // Calculate time remaining
                        const diffMin = prayerTimeInMin - currentTimeInMin;
                        const hours = Math.floor(diffMin / 60);
                        const minutes = diffMin % 60;
                        
                        let remaining = '';
                        if (hours > 0) {
                            remaining = `${hours} ساعة و ${minutes} دقيقة`;
                        } else {
                            remaining = `${minutes} دقيقة`;
                        }
                        
                        this.state.timeRemaining = remaining;
                        return {name: prayer, time: time};
                    }
                }
                
                // Next day's Fajr
                const fajrTimeInMin = timeToMinutes(this.state.prayerTimes.fajr);
                const diffMin = (24 * 60) - currentTimeInMin + fajrTimeInMin;
                const hours = Math.floor(diffMin / 60);
                const minutes = diffMin % 60;
                this.state.timeRemaining = `${hours} ساعة و ${minutes} دقيقة`;
                
                return {name: 'fajr', time: this.state.prayerTimes.fajr};
            },
            
            
            
            handleLogoUpload(event) {
                const file = event.target.files[0];
                if (file && file.type.startsWith('image/')) {
                    const reader = new FileReader();
                    reader.onload = (e) => {
                        this.state.customLogo = e.target.result;
                        localStorage.setItem('customLogo', e.target.result);
                        alert('✅ Logo uploaded successfully!');
                        this.render();
                    };
                    reader.readAsDataURL(file);
                } else {
                    alert('Please upload a valid image file (PNG, JPG, etc.)');
                }
            },
            
            shareApp() {
                const shareData = {
                    title: 'أبو داود - Islamic Prayer App',
                    text: 'Check out this beautiful Islamic prayer times app for Saida, Lebanon! Features: Daily prayer times, Adhan audio, Athkar, and more. Made by Abu Dawod.',
                    url: window.location.href
                };
                
                if (navigator.share) {
                    navigator.share(shareData)
                        .then(() => {
                            document.getElementById('shareStatus').textContent = '✅ Shared successfully!';
                            setTimeout(() => {
                                document.getElementById('shareStatus').textContent = '';
                            }, 3000);
                        })
                        .catch((error) => {
                            if (error.name !== 'AbortError') {
                                this.copyLink();
                            }
                        });
                } else {
                    this.copyLink();
                }
            },
            
            copyLink() {
                const link = window.location.href;
                
                if (navigator.clipboard && navigator.clipboard.writeText) {
                    navigator.clipboard.writeText(link)
                        .then(() => {
                            document.getElementById('shareStatus').textContent = '✅ Link copied to clipboard!';
                            setTimeout(() => {
                                document.getElementById('shareStatus').textContent = '';
                            }, 3000);
                        })
                        .catch(() => {
                            this.fallbackCopyLink(link);
                        });
                } else {
                    this.fallbackCopyLink(link);
                }
            },
            
            fallbackCopyLink(link) {
                const textArea = document.createElement('textarea');
                textArea.value = link;
                textArea.style.position = 'fixed';
                textArea.style.left = '-999999px';
                document.body.appendChild(textArea);
                textArea.focus();
                textArea.select();
                
                try {
                    document.execCommand('copy');
                    document.getElementById('shareStatus').textContent = '✅ Link copied! Share it with WhatsApp, Telegram, etc.';
                    setTimeout(() => {
                        document.getElementById('shareStatus').textContent = '';
                    }, 5000);
                } catch (err) {
                    document.getElementById('shareStatus').textContent = '⚠️ Please manually copy: ' + link;
                }
                
                document.body.removeChild(textArea);
            },
            
            handleAudioUpload(event) {
                const file = event.target.files[0];
                if (file && file.type.startsWith('audio/')) {
                    const audioUrl = URL.createObjectURL(file);
                    this.state.uploadedAudio = audioUrl;
                    alert('✅ Audio file uploaded successfully! Click Play to listen.');
                    this.render();
                } else {
                    alert('Please upload a valid audio file (MP3, WAV, etc.)');
                }
            },
            
            playAdhan() {
                if(this.state.isPlayingAdhan && this.state.currentAudio) {
                    this.state.currentAudio.pause();
                    this.state.currentAudio.currentTime = 0;
                    this.state.isPlayingAdhan = false;
                    this.state.currentAudio = null;
                    this.render();
                    return;
                }
                
                // Use uploaded audio, or selected muadhun URL
                let audioSource = this.state.uploadedAudio || this.muadhuns[this.state.selectedMuadhun].url;
                
                if (!audioSource) {
                    alert('Please upload an MP3 file first, or make sure the MP3 files are in the same folder as this HTML file.');
                    return;
                }
                
                const audio = new Audio(audioSource);
                
                audio.addEventListener('error', (e) => {
                    console.error('Audio error:', e);
                    alert('⚠️ Cannot load audio file.\n\n✅ INSTRUCTIONS:\n1. Save this HTML file in your Downloads folder\n2. Make sure your MP3 files are in the same folder\n3. Or use the "Upload Audio" button to select any MP3 file');
                    this.state.isPlayingAdhan = false;
                    this.render();
                });
                
                audio.play().then(() => {
                    console.log('✅ Audio playing!');
                    this.state.isPlayingAdhan = true;
                    this.state.currentAudio = audio;
                    this.render();
                }).catch((err) => {
                    console.error('Play error:', err);
                    if (err.name === 'NotAllowedError') {
                        alert('Browser blocked audio! Click anywhere on the page first, then try again.');
                    } else {
                        alert('⚠️ Cannot play audio.\n\n✅ SOLUTION: Use the "Upload Audio" button to select your MP3 file manually.');
                    }
                    this.state.isPlayingAdhan = false;
                    this.render();
                });
                
                audio.onended = () => {
                    this.state.isPlayingAdhan = false;
                    this.state.currentAudio = null;
                    this.render();
                };
            },
            
            async toggleNotifications() {
                if(!this.state.notificationsEnabled) {
                    if('Notification' in window) {
                        const perm = await Notification.requestPermission();
                        if(perm === 'granted') {
                            this.state.notificationsEnabled = true;
                            localStorage.setItem('notificationsEnabled', 'true');
                        }
                    }
                } else {
                    this.state.notificationsEnabled = false;
                    localStorage.setItem('notificationsEnabled', 'false');
                }
                this.render();
            },
            
            icon(name) {
                const icons = {
                    moon: '<path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path>',
                    bell: '<path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"></path><path d="M13.73 21a2 2 0 0 1-3.46 0"></path>',
                    clock: '<circle cx="12" cy="12" r="10"></circle><polyline points="12 6 12 12 16 14"></polyline>',
                    sun: '<circle cx="12" cy="12" r="5"></circle><line x1="12" y1="1" x2="12" y2="3"></line><line x1="12" y1="21" x2="12" y2="23"></line><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line><line x1="1" y1="12" x2="3" y2="12"></line><line x1="21" y1="12" x2="23" y2="12"></line><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line>',
                    book: '<path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20"></path><path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z"></path>',
                    heart: '<path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"></path>',
                    wifi: '<path d="M5 12.55a11 11 0 0 1 14.08 0"></path><path d="M1.42 9a16 16 0 0 1 21.16 0"></path><path d="M8.53 16.11a6 6 0 0 1 6.95 0"></path><line x1="12" y1="20" x2="12.01" y2="20"></line>',
                    wifioff: '<line x1="1" y1="1" x2="23" y2="23"></line><path d="M16.72 11.06A10.94 10.94 0 0 1 19 12.55"></path><path d="M5 12.55a10.94 10.94 0 0 1 5.17-2.39"></path><path d="M10.71 5.05A16 16 0 0 1 22.58 9"></path><path d="M1.42 9a15.91 15.91 0 0 1 4.7-2.88"></path><path d="M8.53 16.11a6 6 0 0 1 6.95 0"></path><line x1="12" y1="20" x2="12.01" y2="20"></line>',
                    refresh: '<polyline points="23 4 23 10 17 10"></polyline><polyline points="1 20 1 14 7 14"></polyline><path d="M3.51 9a9 9 0 0 1 14.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0 0 20.49 15"></path>',
                    pin: '<path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z"></path><circle cx="12" cy="10" r="3"></circle>'
                };
                return `<svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">${icons[name]}</svg>`;
            },
            
            
            translatePrayerName(prayer) {
                const names = {
                    fajr: 'الفجر',
                    sunrise: 'الشروق',
                    dhuhr: 'الظهر',
                    asr: 'العصر',
                    maghrib: 'المغرب',
                    isha: 'العشاء'
                };
                return names[prayer] || prayer;
            },
            
            render() {
                const s = this.state;
                const next = this.getNextPrayer();
                
                document.getElementById('app').innerHTML = `
                    <div class="min-h-screen">
                        <!-- App Header with Logo -->
                        <div class="bg-gradient-to-r from-green-600 to-blue-600 text-white p-6 shadow-lg">
                            <!-- App Logo -->
                            <div class="app-logo arabic-text">أد</div>
                            
                            <div class="flex items-center justify-between flex-wrap gap-4">
                                <div class="flex items-center space-x-3">
                                    <div>
                                        <h1 class="text-2xl font-bold arabic-text">أبو داود</h1>
                                        <p class="text-sm text-green-100">Prayer Times - Saida, Lebanon</p>
                                    </div>
                                </div>
                                <div class="text-right">
                                    <div class="text-2xl font-bold">${s.currentTime.toLocaleTimeString('en-US', {hour: 'numeric', minute: '2-digit', hour12: true})}</div>
                                    <div class="text-sm opacity-90">${s.currentTime.toLocaleDateString()}</div>
                                    ${s.currentHijriDate ? `<div class="text-xs opacity-75 mt-1 arabic-text">${s.currentHijriDate}</div>` : ''}
                                </div>
                            </div>
                            
                            <!-- Custom Logo Section -->
                            ${s.customLogo ? `
                                <div class="mt-6 text-center">
                                    <img src="${s.customLogo}" class="custom-logo-img mx-auto" style="max-height: 120px; border-radius: 15px; box-shadow: 0 5px 15px rgba(0,0,0,0.2);" alt="App Logo">
                                </div>
                            ` : ''}
                        </div>

                        <!-- Tabs -->
                        <div class="bg-white shadow-sm">
                            <div class="flex overflow-x-auto" dir="rtl">
                                ${[
                                    {id: 'prayer', label: 'الصلاة'},
                                    {id: 'adhan', label: 'الأذان'},
                                    {id: 'athkar', label: 'الأذكار'},
                                    {id: 'tawba', label: 'التوبة'},
                                    {id: 'profile', label: '👤 الملف'}
                                ].map(tab => `
                                    <button onclick="app.state.activeTab='${tab.id}'; app.render()" 
                                        class="tab-button flex-1 flex items-center justify-center space-x-2 py-4 px-6 border-b-2 transition-colors whitespace-nowrap ${
                                            s.activeTab === tab.id ? 'border-green-500 text-green-600 bg-green-50' : 'border-transparent text-gray-500 hover:text-gray-700'
                                        }">
                                        <span class="font-medium arabic-text">${tab.label}</span>
                                    </button>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Made by Abu Dawod Banner -->
                        <div class="p-6 pb-4">
                            <div class="bg-gradient-to-r from-green-600 to-blue-600 text-white p-4 rounded-xl shadow-lg text-center">
                                <p class="text-lg font-semibold arabic-text">صُنع بـ ❤️ من قبل أبو داود</p>
                                <p class="text-sm opacity-90">May Allah accept this work and make it beneficial for all Muslims</p>
                            </div>
                        </div>

                        <!-- Content -->
                        <div class="p-6 pt-0">
                            ${s.activeTab === 'prayer' ? this.renderPrayer(next) : ''}
                            ${s.activeTab === 'adhan' ? this.renderAdhan() : ''}
                            ${s.activeTab === 'athkar' ? this.renderAthkar() : ''}
                            ${s.activeTab === 'tawba' ? this.renderTawba() : ''}
                            ${s.activeTab === 'profile' ? this.renderProfile() : ''}
                        </div>
                    </div>
                `;
            },
            
            renderProfile() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <!-- Developer Profile Header -->
                        <div class="bg-gradient-to-r from-green-600 via-emerald-600 to-blue-600 text-white p-8 rounded-xl shadow-2xl">
                            <div class="text-center">
                                <!-- Profile Picture Upload Area -->
                                <div class="mb-6">
                                    <div class="w-32 h-32 mx-auto rounded-full bg-white bg-opacity-20 flex items-center justify-center cursor-pointer hover:bg-opacity-30 transition-all overflow-hidden border-4 border-white shadow-lg" onclick="document.getElementById('logoUpload').click()">
                                        ${s.customLogo ? `
                                            <img src="${s.customLogo}" class="w-full h-full object-cover" alt="Profile">
                                        ` : `
                                            <svg class="w-16 h-16 text-white opacity-75" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"></path>
                                            </svg>
                                        `}
                                    </div>
                                    <input 
                                        type="file" 
                                        id="logoUpload"
                                        accept="image/*" 
                                        onchange="app.handleLogoUpload(event)"
                                        style="display: none;"
                                    />
                                    <p class="text-sm mt-2 opacity-75">Click to upload profile photo</p>
                                </div>
                                
                                <h1 class="text-4xl font-bold mb-2 arabic-text">أبو داود</h1>
                                <p class="text-xl mb-2">Abu Dawod</p>
                                <p class="text-lg opacity-90">Islamic App Developer</p>
                                <div class="flex items-center justify-center gap-2 mt-4">
                                    <span class="px-4 py-1 bg-white bg-opacity-20 rounded-full text-sm">Saida, Lebanon 🇱🇧</span>
                                </div>
                            </div>
                        </div>

                        <!-- About This Project -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">📱 About This Application</h2>
                            <div class="space-y-4 text-gray-700">
                                <p class="leading-relaxed">
                                    <strong class="text-green-600">Islamic Prayer Times & Athkar App</strong> is a comprehensive web application designed to help Muslims stay connected with their daily prayers and remembrance of Allah.
                                </p>
                                <p class="leading-relaxed">
                                    This app provides accurate prayer times for Saida, Lebanon, beautiful Adhan audio, daily Athkar, and reminders for Tawba (repentance).
                                </p>
                            </div>
                        </div>

                        <!-- Features -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">✨ Key Features</h2>
                            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                                ${[
                                    { icon: '🕌', title: 'Daily Prayer Times', desc: 'Accurate times for Saida updated automatically' },
                                    { icon: '🔔', title: 'Prayer Notifications', desc: 'Get notified when it\'s time to pray' },
                                    { icon: '🎵', title: 'Beautiful Adhan', desc: 'Multiple Adhan styles with audio playback' },
                                    { icon: '📿', title: 'Daily Athkar', desc: 'Morning, evening, and general remembrances' },
                                    { icon: '💜', title: 'Tawba Reminders', desc: 'Encouragement for repentance and forgiveness' },
                                    { icon: '🌙', title: 'Hijri Calendar', desc: 'Display Islamic date alongside Gregorian' },
                                    { icon: '📱', title: 'Installable App', desc: 'Works like a native mobile application' },
                                    { icon: '🔄', title: 'Auto-Updates', desc: 'Prayer times update daily automatically' }
                                ].map(f => `
                                    <div class="flex items-start space-x-3 p-3 bg-green-50 rounded-lg">
                                        <span class="text-2xl">${f.icon}</span>
                                        <div>
                                            <h3 class="font-semibold text-gray-800">${f.title}</h3>
                                            <p class="text-sm text-gray-600">${f.desc}</p>
                                        </div>
                                    </div>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Technologies Used -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">🛠️ Technologies Used</h2>
                            <div class="flex flex-wrap gap-3">
                                ${['HTML5', 'CSS3', 'JavaScript', 'Tailwind CSS', 'AlAdhan API', 'Web Audio API', 'Local Storage', 'PWA'].map(tech => `
                                    <span class="px-4 py-2 bg-gradient-to-r from-green-100 to-blue-100 text-gray-800 rounded-full text-sm font-semibold border border-green-200">${tech}</span>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Statistics -->
                        <div class="bg-gradient-to-r from-purple-50 to-pink-50 p-6 rounded-xl shadow-lg border-2 border-purple-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">📊 App Statistics</h2>
                            <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
                                ${[
                                    { num: '5', label: 'Daily Prayers' },
                                    { num: '10+', label: 'Athkar Duas' },
                                    { num: '4', label: 'Adhan Styles' },
                                    { num: '100%', label: 'Free & Open' }
                                ].map(stat => `
                                    <div class="text-center p-4 bg-white rounded-lg shadow-sm">
                                        <div class="text-3xl font-bold text-green-600">${stat.num}</div>
                                        <div class="text-sm text-gray-600 mt-1">${stat.label}</div>
                                    </div>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Contact & Share -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">🤝 Share This App</h2>
                            <p class="text-gray-700 mb-4">Help spread the benefit! Share this Islamic prayer app with your community:</p>
                            <div class="space-y-3">
                                <button onclick="app.shareApp()" 
                                    class="w-full px-6 py-3 bg-gradient-to-r from-green-500 to-blue-500 text-white rounded-lg font-semibold hover:from-green-600 hover:to-blue-600 transition-all shadow-md">
                                    📤 Share App
                                </button>
                                <button onclick="app.copyLink()" 
                                    class="w-full px-6 py-3 bg-gray-100 text-gray-800 rounded-lg font-semibold hover:bg-gray-200 transition-all">
                                    📋 Copy App Link
                                </button>
                                <div class="text-center text-sm text-gray-500 mt-2">
                                    <p id="shareStatus"></p>
                                </div>
                            </div>
                        </div>

                        <!-- Credits & License -->
                        <div class="bg-gradient-to-r from-green-600 to-blue-600 text-white p-6 rounded-xl shadow-lg text-center">
                            <h2 class="text-2xl font-bold mb-3 arabic-text">بارك الله فيك</h2>
                            <p class="text-lg mb-4">May Allah bless this work and make it a source of continuous reward (Sadaqah Jariyah)</p>
                            <div class="text-sm opacity-90">
                                <p>Developed with ❤️ by <strong class="arabic-text">أبو داود</strong></p>
                                <p class="mt-2">Version 1.0.0 | 2025</p>
                                <p class="mt-2">Free for all Muslims to use and benefit from</p>
                            </div>
                        </div>

                        ${s.customLogo ? `
                            <div class="bg-white p-4 rounded-lg shadow-sm border border-gray-200 text-center">
                                <button 
                                    onclick="app.state.customLogo = null; localStorage.removeItem('customLogo'); app.render();"
                                    class="px-6 py-2 bg-red-50 hover:bg-red-100 text-red-600 rounded-lg font-medium transition-colors">
                                    🗑️ Remove Profile Photo
                                </button>
                            </div>
                        ` : ''}
                    </div>
                `;
            },
            
            renderSettings() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-purple-500 to-indigo-500 text-white p-6 rounded-xl shadow-lg">
                            <h2 class="text-2xl font-bold mb-2">⚙️ Settings</h2>
                            <p class="text-lg opacity-90">Upload your program image</p>
                        </div>

                        <!-- Upload Photo Only -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-lg font-semibold mb-4 text-center">📸 Program Image</h3>
                            
                            <div class="custom-logo-container" onclick="document.getElementById('logoUpload').click()">
                                ${s.customLogo ? `
                                    <img src="${s.customLogo}" class="custom-logo-img" alt="Program Image">
                                ` : `
                                    <div class="text-center text-gray-400">
                                        <svg class="w-16 h-16 mx-auto mb-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"></path>
                                        </svg>
                                        <p class="text-sm font-semibold">Click to upload photo</p>
                                    </div>
                                `}
                            </div>
                            
                            <input 
                                type="file" 
                                id="logoUpload"
                                accept="image/*" 
                                onchange="app.handleLogoUpload(event)"
                                style="display: none;"
                            />
                            
                            ${s.customLogo ? `
                                <button 
                                    onclick="app.state.customLogo = null; localStorage.removeItem('customLogo'); app.render();"
                                    class="w-full mt-4 px-4 py-3 bg-red-50 hover:bg-red-100 text-red-600 rounded-lg font-semibold transition-colors">
                                    🗑️ Remove Photo
                                </button>
                            ` : ''}
                        </div>
                    </div>
                `;
            },
            
            renderPrayer(next) {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-green-500 to-blue-500 text-white p-6 rounded-xl shadow-lg transform transition-all hover:shadow-2xl">
                            <div class="text-center">
                                <h3 class="text-lg font-semibold opacity-90 mb-2">الصلاة القادمة</h3>
                                <h2 class="text-4xl font-bold mb-2 arabic-text">${this.translatePrayerName(next.name)}</h2>
                                <p class="text-2xl opacity-90 mb-3">${next.time}</p>
                                
                                <!-- Time Remaining -->
                                <div class="bg-white bg-opacity-20 rounded-lg p-4 mt-4">
                                    <p class="text-sm opacity-90 mb-1">الوقت المتبقي</p>
                                    <p class="text-3xl font-bold arabic-text">${s.timeRemaining}</p>
                                </div>
                            </div>
                        </div>

                        <div class="bg-white p-4 rounded-lg shadow-sm border border-gray-200">
                            <div class="flex items-center justify-between mb-3">
                                <div class="flex items-center space-x-3 space-x-reverse">
                                    ${this.icon('bell')}
                                    <span class="font-medium arabic-text">تنبيهات الصلاة</span>
                                </div>
                                <button onclick="app.toggleNotifications()" 
                                    class="relative inline-flex h-6 w-11 items-center rounded-full transition-colors ${s.notificationsEnabled ? 'bg-green-600' : 'bg-gray-200'}">
                                    <span class="inline-block h-4 w-4 transform rounded-full bg-white transition-transform ${s.notificationsEnabled ? 'translate-x-6' : 'translate-x-1'}"></span>
                                </button>
                            </div>
                            <div class="flex items-center justify-between text-sm pt-3 border-t border-gray-100">
                                <div class="flex items-center space-x-2 space-x-reverse">
                                    ${s.connectionStatus === 'connected' ? `${this.icon('wifi')}<span class="text-green-600 arabic-text">متصل بصيدا</span>` : 
                                      s.connectionStatus === 'connecting' ? `<div class="animate-spin">${this.icon('refresh')}</div><span class="text-blue-600 arabic-text">جاري الاتصال...</span>` :
                                      `${this.icon('wifioff')}<span class="text-red-600 arabic-text">غير متصل</span>`}
                                </div>
                                ${s.lastUpdated ? `<span class="text-gray-500 arabic-text">آخر تحديث: ${s.lastUpdated.toLocaleTimeString('ar-LB', {hour: 'numeric', minute: '2-digit', hour12: true})}</span>` : ''}
                            </div>
                            <button onclick="app.fetchPrayerTimes()" ${s.isLoading ? 'disabled' : ''}
                                class="w-full mt-3 px-4 py-2 bg-blue-50 hover:bg-blue-100 text-blue-600 rounded-lg font-medium transition-colors disabled:opacity-50 arabic-text">
                                ${s.isLoading ? '🔄 جاري التحديث...' : '🔄 تحديث أوقات الصلاة'}
                            </button>
                        </div>

                        <div class="bg-white rounded-lg shadow-sm border border-gray-200 overflow-hidden">
                            <h3 class="text-lg font-semibold p-4 border-b border-gray-200 bg-gray-50 arabic-text text-right">أوقات الصلاة اليوم</h3>
                            <div class="divide-y divide-gray-100">
                                ${Object.entries(s.prayerTimes).map(([p,t]) => `
                                    <div class="flex items-center justify-between p-4 hover:bg-gray-50 transition-colors" dir="rtl">
                                        <div class="flex items-center space-x-3 space-x-reverse">
                                            <div class="w-3 h-3 rounded-full ${p === next.name ? 'bg-green-500 animate-pulse' : 'bg-gray-300'}"></div>
                                            <span class="font-medium text-gray-700 arabic-text text-lg">${this.translatePrayerName(p)}</span>
                                        </div>
                                        <span class="text-lg font-semibold text-gray-900">${t}</span>
                                    </div>
                                `).join('')}
                            </div>
                        </div>
                    </div>
                `;
            },
            
            renderAdhan() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-amber-500 to-orange-500 text-white p-6 rounded-xl shadow-lg">
                            <div class="flex items-center space-x-3 mb-4">
                                ${this.icon('sun')}
                                <h2 class="text-2xl font-bold">Adhan Player</h2>
                            </div>
                            <p class="text-lg opacity-90">Listen to the beautiful call to prayer</p>
                        </div>

                        <!-- Instructions -->
                        <div class="bg-blue-50 p-4 rounded-lg border-2 border-blue-200">
                            <h3 class="font-bold text-blue-800 mb-2">📋 Instructions:</h3>
                            <ol class="text-sm text-blue-700 space-y-1 list-decimal list-inside">
                                <li>Save this HTML file in your <strong>Downloads folder</strong></li>
                                <li>Your MP3 files should be in the <strong>same folder</strong></li>
                                <li>Or click "Choose File" below to upload any MP3</li>
                            </ol>
                        </div>

                        <!-- Upload Audio -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-lg font-semibold mb-4">📁 Upload Your Adhan MP3</h3>
                            <input
                                type="file"
                                accept="audio/*"
                                onchange="app.handleAudioUpload(event)"
                                class="block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-green-50 file:text-green-700 hover:file:bg-green-100 cursor-pointer"
                            />
                            ${s.uploadedAudio ? '<p class="text-green-600 text-sm mt-2">✅ Audio uploaded! Ready to play.</p>' : ''}
                        </div>

                        <!-- Choose Adhan -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-lg font-semibold mb-4">Choose Adhan Source</h3>
                            <div class="space-y-3">
                                ${Object.entries(this.muadhuns).map(([k,m]) => `
                                    <div onclick="app.state.selectedMuadhun='${k}'; app.render()" 
                                        class="p-4 rounded-lg border-2 cursor-pointer transition-all transform hover:scale-102 ${
                                            s.selectedMuadhun === k ? 'border-amber-500 bg-amber-50 shadow-md' : 'border-gray-200 hover:border-gray-300'
                                        }">
                                        <div class="flex items-center justify-between">
                                            <div>
                                                <h4 class="font-semibold text-gray-800">${m.name}</h4>
                                                <p class="text-sm text-gray-600">${m.desc}</p>
                                            </div>
                                            <div class="w-4 h-4 rounded-full border-2 ${s.selectedMuadhun === k ? 'border-amber-500 bg-amber-500' : 'border-gray-300'}"></div>
                                        </div>
                                    </div>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Play Button -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <div class="text-center space-y-4">
                                <h3 class="text-lg font-semibold">${this.muadhuns[s.selectedMuadhun].name}</h3>
                                <button onclick="app.playAdhan()" 
                                    class="px-8 py-4 rounded-full text-white font-semibold text-lg transition-all transform hover:scale-105 shadow-lg ${
                                        s.isPlayingAdhan ? 'bg-red-500 hover:bg-red-600 animate-pulse' : 'bg-gradient-to-r from-green-500 to-blue-500 hover:from-green-600 hover:to-blue-600'
                                    }">
                                    ${s.isPlayingAdhan ? '⏸️ Stop Adhan' : '▶️ Play Adhan'}
                                </button>
                                ${s.isPlayingAdhan ? `
                                    <div class="bg-green-50 p-4 rounded-lg border border-green-200 animate-pulse">
                                        <p class="text-green-700 font-medium">🎵 Playing Adhan...</p>
                                        <p class="text-sm text-gray-600 mt-1">${s.uploadedAudio ? 'Your uploaded audio' : this.muadhuns[s.selectedMuadhun].name}</p>
                                    </div>
                                ` : ''}
                                <p class="text-sm text-gray-500 mt-2">🔊 Make sure your volume is on</p>
                            </div>
                        </div>
                    </div>
                `;
            },
            
            renderAthkar() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-white p-4 rounded-lg shadow-sm border border-gray-200">
                            <div class="flex space-x-2 overflow-x-auto">
                                ${Object.keys(this.athkar).map(cat => `
                                    <button onclick="app.state.selectedAthkar='${cat}'; app.render()" 
                                        class="px-4 py-2 rounded-lg font-medium capitalize transition-all whitespace-nowrap ${
                                            s.selectedAthkar === cat ? 'bg-green-500 text-white shadow-md' : 'bg-gray-100 text-gray-700 hover:bg-gray-200'
                                        }">
                                        ${cat}
                                    </button>
                                `).join('')}
                            </div>
                        </div>

                        ${this.athkar[s.selectedAthkar].map(d => `
                            <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200 transform transition-all hover:shadow-lg">
                                <div class="text-right mb-4">
                                    <p class="text-2xl text-green-700 leading-relaxed">${d.ar}</p>
                                </div>
                                <div class="space-y-2">
                                    <p class="text-gray-600 italic text-sm">${d.tr}</p>
                                    <p class="text-gray-800 font-medium">${d.en}</p>
                                </div>
                            </div>
                        `).join('')}
                    </div>
                `;
            },
            
            renderTawba() {
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-purple-500 to-pink-500 text-white p-6 rounded-xl shadow-lg">
                            <div class="flex items-center space-x-3 space-x-reverse mb-4" dir="rtl">
                                ${this.icon('heart')}
                                <h2 class="text-2xl font-bold arabic-text">التوبة - الاستغفار</h2>
                            </div>
                            <p class="text-lg opacity-90 arabic-text text-right leading-relaxed">
                                "وَمَن تَابَ بَعْدَ ظُلْمِهِ وَأَصْلَحَ فَإِنَّ اللَّهَ يَتُوبُ عَلَيْهِ ۗ إِنَّ اللَّهَ غَفُورٌ رَّحِيمٌ" - سورة المائدة 39
                            </p>
                        </div>

                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-xl font-semibold mb-4 text-center arabic-text">الاستغفار اليومي</h3>
                            <div class="text-center">
                                <div class="text-4xl font-bold text-green-600 mb-4 arabic-text leading-relaxed">
                                    أَسْتَغْفِرُ اللَّهَ الْعَظِيمَ
                                </div>
                                <p class="text-gray-600 mb-2 italic">Astaghfiru Allah al-Adheem</p>
                                <p class="text-sm text-gray-500 mb-4 arabic-text">أستغفر الله العظيم وأتوب إليه</p>
                            </div>
                        </div>

                        <div class="space-y-4">
                            <h3 class="text-xl font-semibold text-gray-700 arabic-text text-right">أدعية التوبة والاستغفار</h3>
                            
                            <div class="bg-gradient-to-r from-blue-50 to-purple-50 p-6 rounded-lg border-l-4 border-purple-400">
                                <p class="text-2xl text-gray-800 arabic-text text-right leading-loose mb-3">
                                    اللَّهُمَّ إِنَّكَ عَفُوٌّ كَرِيمٌ تُحِبُّ الْعَفْوَ فَاعْفُ عَنَّا
                                </p>
                                <p class="text-sm text-gray-600 italic">Allahumma innaka 'afuwwun karimun tuhibbul-'afwa fa'fu 'anna</p>
                                <p class="text-sm text-gray-700 mt-2 arabic-text text-right">
                                    اللهم إنك عفو كريم تحب العفو فاعف عنا
                                </p>
                            </div>

                            <div class="bg-gradient-to-r from-green-50 to-blue-50 p-6 rounded-lg border-l-4 border-green-400">
                                <p class="text-2xl text-gray-800 arabic-text text-right leading-loose mb-3">
                                    رَبَّنَا آتِنَا فِي الدُّنْيَا حَسَنَةً وَفِي الْآخِرَةِ حَسَنَةً وَقِنَا عَذَابَ النَّارِ
                                </p>
                                <p class="text-sm text-gray-600 italic">Rabbana atina fid-dunya hasanatan wa fil-akhirati hasanatan wa qina 'adhaban-nar</p>
                                <p class="text-sm text-gray-700 mt-2 arabic-text text-right">
                                    اللهم آتنا في الدنيا حسنة وفي الآخرة حسنة وقنا عذاب النار
                                </p>
                            </div>

                            <div class="bg-gradient-to-r from-purple-50 to-pink-50 p-6 rounded-lg border-l-4 border-pink-400">
                                <p class="text-2xl text-gray-800 arabic-text text-right leading-loose mb-3">
                                    اللَّهُمَّ آتِ نَفْسِي تَقْوَاهَا وَزَكِّهَا أَنتَ خَيْرُ مَن زَكَّاهَا أَنتَ وَلِيُّهَا وَمَوْلَاهَا
                                </p>
                                <p class="text-sm text-gray-600 italic">Allahumma ati nafsi taqwaha wa zakkiha anta khayru man zakkaha anta waliyyuha wa mawlaha</p>
                                <p class="text-sm text-gray-700 mt-2 arabic-text text-right">
                                    اللهم آت نفسي تقواها وزكها أنت خير من زكاها أنت وليها ومولاها
                                </p>
                            </div>
                        </div>

                        <div class="space-y-4">
                            <h3 class="text-xl font-semibold text-gray-700 arabic-text text-right">تذكيرات يومية</h3>
                            ${this.tawbaReminders.map(r => `
                                <div class="bg-gradient-to-r from-blue-50 to-purple-50 p-4 rounded-lg border-r-4 border-purple-400 transform transition-all hover:shadow-md" dir="rtl">
                                    <p class="text-gray-700 arabic-text text-right">${r}</p>
                                </div>
                            `).join('')}
                        </div>

                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-xl font-semibold mb-4 arabic-text text-right">دعاء الاستغفار</h3>
                            <div class="border-r-4 border-green-400 pr-4" dir="rtl">
                                <p class="text-2xl text-green-700 mb-3 text-right leading-loose arabic-text">
                                    رَبَّنَا ظَلَمْنَا أَنفُسَنَا وَإِن لَّمْ تَغْفِرْ لَنَا وَتَرْحَمْنَا لَنَكُونَنَّ مِنَ الْخَاسِرِينَ
                                </p>
                                <p class="text-sm text-gray-600 italic mb-1">
                                    Rabbana zalamna anfusana wa in lam taghfir lana wa tarhamna lanakoonanna minal-khasireen
                                </p>
                                <p class="text-sm text-gray-700 arabic-text text-right">
                                    "رَبَّنَا ظَلَمْنَا أَنفُسَنَا وَإِن لَّمْ تَغْفِرْ لَنَا وَتَرْحَمْنَا لَنَكُونَنَّ مِنَ الْخَاسِرِينَ"
                                </p>
                            </div>
                        </div>
                    </div>
                `;
            },
            
            init() {
                this.fetchPrayerTimes();
                setInterval(() => {
                    this.state.currentTime = new Date();
                    this.render();
                }, 1000);
                setInterval(() => this.fetchPrayerTimes(), 21600000);
                setInterval(() => {
                    const now = new Date();
                    if(now.getHours() === 0 && now.getMinutes() === 0) {
                        this.fetchPrayerTimes();
                    }
                }, 60000);
                this.render();
            }
        };
        
        window.app = app;
        app.init();
    </script>
</body>
</html><!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <meta name="author" content="Abu Dawod">
    <meta name="description" content="Islamic Prayer Times App for Saida, Lebanon - Made by Abu Dawod">
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="apple-mobile-web-app-title" content="أبو داود">
    <meta name="application-name" content="أبو داود">
    <meta name="theme-color" content="#16a34a">
    
    <!-- PWA Manifest -->
    <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoi2KPYqNmIINiv2KfZiNivIiwic2hvcnRfbmFtZSI6Itin2KjZiCDYr9in2YjYryIsInN0YXJ0X3VybCI6Ii4vaW5kZXguaHRtbCIsImRpc3BsYXkiOiJzdGFuZGFsb25lIiwiYmFja2dyb3VuZF9jb2xvciI6IiNmZmZmZmYiLCJ0aGVtZV9jb2xvciI6IiMxNmEzNGEiLCJpY29ucyI6W3sic3JjIjoiZGF0YTppbWFnZS9zdmcreG1sO2Jhc2U2NCxQSE4yWnlCNGJXeHVjejBpYUhSMGNEb3ZMM2QzZHk1M015NXZjbWN2TWpBd01DOXpkbWNpSUhacFpYZENiM2c5SWpBZ01DQXlOQ0F5TkNJK1BHTnBjbU5zWlNCamVEMGlNVElpSUdONVBTSXhNaUlnY2owSU5pSWdabWxzYkQwaUl6RTJZVE0wWVNJdlBqd3ZjM1puUGc9PSIsInNpemVzIjoiNTEyeDUxMiIsInR5cGUiOiJpbWFnZS9zdmcreG1sIn1dfQ==">
    
    <!-- Favicon / App Icon -->
    <link rel="icon" type="image/svg+xml" href="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI+PGNpcmNsZSBjeD0iMTIiIGN5PSIxMiIgcj0iNiIgZmlsbD0iIzE2YTM0YSIvPjwvc3ZnPg==">
    
    <title>أبو داود - Prayer Times</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .animate-pulse { animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite; }
        @keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: .5; } }
        .animate-spin { animation: spin 1s linear infinite; }
        @keyframes spin { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }
        .transition-all { transition: all 0.3s ease; }
        .transform:hover { transform: scale(1.02); }
        .tab-button { transition: all 0.3s ease; }
        .tab-button:hover { background-color: rgba(0,0,0,0.05); }
    </style>
</head>
<body class="bg-gradient-to-br from-green-50 to-blue-50 min-h-screen">
    <div id="app"></div>

    <script>
        const app = {
            state: {
                currentTime: new Date(),
                activeTab: 'prayer',
                notificationsEnabled: localStorage.getItem('notificationsEnabled') === 'true',
                selectedAthkar: 'morning',
                selectedMuadhun: 'local1',
                isPlayingAdhan: false,
                currentAudio: null,
                uploadedAudio: null,
                prayerTimes: {
                    fajr: '05:30',
                    sunrise: '06:45',
                    dhuhr: '12:15',
                    asr: '15:45',
                    maghrib: '18:20',
                    isha: '19:45'
                },
                isLoading: false,
                lastUpdated: null,
                connectionStatus: 'loading',
                currentHijriDate: '',
                customLogo: localStorage.getItem('customLogo') || null
            },
            
            SAIDA: { latitude: 33.5631, longitude: 35.3708, city: 'Saida', country: 'Lebanon' },
            
            muadhuns: {
                local1: { name: 'Your Adhan 1 (haii3alslah936584)', desc: 'Local MP3 file from Downloads', url: './haii3alslah936584 (1).mp3' },
                local2: { name: 'Your Adhan 2 (haii3alslah2336584)', desc: 'Local MP3 file from Downloads', url: './haii3alslah2336584.mp3' },
                upload: { name: 'Upload Your Own', desc: 'Click to upload any MP3 file', url: null }
            },
            
            athkar: {
                morning: [
                    { ar: 'اللَّهُمَّ أَعِنِّي عَلَى ذِكْرِكَ وَشُكْرِكَ وَحُسْنِ عِبَادَتِكَ', tr: 'Allahumma a\'inni ala dhikrika wa shukrika wa husni ibadatika', en: 'O Allah, help me to remember You, to thank You, and to worship You in the best manner.' },
                    { ar: 'أَصْبَحْنَا وَأَصْبَحَ الْمُلْكُ لِلَّهِ', tr: 'Asbahna wa asbaha al-mulku lillahi', en: 'We have entered the morning and the whole kingdom belongs to Allah.' },
                    { ar: 'اللَّهُمَّ بِكَ أَصْبَحْنَا وَبِكَ أَمْسَيْنَا وَبِكَ نَحْيَا وَبِكَ نَمُوتُ وَإِلَيْكَ النُّشُورُ', tr: 'Allahumma bika asbahna wa bika amsayna wa bika nahya wa bika namutu wa ilayka an-nushur', en: 'O Allah, by You we enter the morning and by You we enter the evening, by You we live and by You we die, and to You is the resurrection.' }
                ],
                evening: [
                    { ar: 'أَمْسَيْنَا وَأَمْسَى الْمُلْكُ لِلَّهِ', tr: 'Amsayna wa amsa al-mulku lillahi', en: 'We have entered the evening and the whole kingdom belongs to Allah.' },
                    { ar: 'اللَّهُمَّ إِنِّي أَعُوذُ بِكَ مِنْ عَذَابِ جَهَنَّمَ وَمِنْ عَذَابِ الْقَبْرِ', tr: 'Allahumma inni a\'udhu bika min adhabi jahannama wa min adhabi al-qabr', en: 'O Allah, I seek refuge in You from the punishment of Hell and from the punishment of the grave.' }
                ],
                general: [
                    { ar: 'سُبْحَانَ اللَّهِ وَبِحَمْدِهِ', tr: 'Subhan Allahi wa bihamdihi', en: 'Glory be to Allah and praise be to Him.' },
                    { ar: 'لَا إِلَهَ إِلَّا اللَّهُ وَحْدَهُ لَا شَرِيكَ لَهُ', tr: 'La ilaha illa Allah wahdahu la sharika lahu', en: 'There is no god but Allah alone, with no partner.' },
                    { ar: 'أَسْتَغْفِرُ اللَّهَ الْعَظِيمَ', tr: 'Astaghfiru Allah al-adheem', en: 'I seek forgiveness from Allah, the Most Great.' }
                ]
            },
            
            tawbaReminders: [
                'Remember to seek Allah\'s forgiveness regularly - He is Most Forgiving, Most Merciful.',
                'Turn to Allah in repentance before it\'s too late. Every moment is a chance for a new beginning.',
                'Allah loves those who turn to Him in repentance and purify themselves.',
                'No sin is too great for Allah\'s forgiveness when you sincerely repent.',
                'Make Istighfar (seeking forgiveness) a daily habit - it brings peace to the heart.'
            ],
            
            async fetchPrayerTimes() {
                this.state.isLoading = true;
                this.state.connectionStatus = 'connecting';
                this.render();
                
                try {
                    const d = new Date();
                    const url = `https://api.aladhan.com/v1/timings/${d.getDate()}-${d.getMonth()+1}-${d.getFullYear()}?latitude=${this.SAIDA.latitude}&longitude=${this.SAIDA.longitude}&method=8`;
                    const res = await fetch(url);
                    const data = await res.json();
                    
                    // Convert 24hr to 12hr format
                    const convert24to12 = (time24) => {
                        const [hours, minutes] = time24.split(':');
                        const h = parseInt(hours);
                        const period = h >= 12 ? 'PM' : 'AM';
                        const h12 = h === 0 ? 12 : h > 12 ? h - 12 : h;
                        return `${h12}:${minutes} ${period}`;
                    };
                    
                    this.state.prayerTimes = {
                        fajr: convert24to12(data.data.timings.Fajr),
                        sunrise: convert24to12(data.data.timings.Sunrise),
                        dhuhr: convert24to12(data.data.timings.Dhuhr),
                        asr: convert24to12(data.data.timings.Asr),
                        maghrib: convert24to12(data.data.timings.Maghrib),
                        isha: convert24to12(data.data.timings.Isha)
                    };
                    
                    this.state.currentHijriDate = `${data.data.date.hijri.day} ${data.data.date.hijri.month.en} ${data.data.date.hijri.year}`;
                    this.state.connectionStatus = 'connected';
                    this.state.lastUpdated = new Date();
                } catch(e) {
                    console.error(e);
                    this.state.connectionStatus = 'offline';
                }
                
                this.state.isLoading = false;
                this.render();
            },
            
            getNextPrayer() {
                const now = new Date();
                const currentHour = now.getHours();
                const currentMin = now.getMinutes();
                const currentTimeInMin = currentHour * 60 + currentMin;
                
                // Convert 12hr format back to minutes for comparison
                const timeToMinutes = (time12) => {
                    const match = time12.match(/(\d+):(\d+)\s*(AM|PM)/i);
                    if (!match) return 0;
                    let hours = parseInt(match[1]);
                    const minutes = parseInt(match[2]);
                    const period = match[3].toUpperCase();
                    
                    if (period === 'PM' && hours !== 12) hours += 12;
                    if (period === 'AM' && hours === 12) hours = 0;
                    
                    return hours * 60 + minutes;
                };
                
                const prayers = [
                    ['fajr', this.state.prayerTimes.fajr],
                    ['dhuhr', this.state.prayerTimes.dhuhr],
                    ['asr', this.state.prayerTimes.asr],
                    ['maghrib', this.state.prayerTimes.maghrib],
                    ['isha', this.state.prayerTimes.isha]
                ];
                
                for(let [prayer, time] of prayers) {
                    const prayerTimeInMin = timeToMinutes(time);
                    if(prayerTimeInMin > currentTimeInMin) {
                        // Calculate time remaining
                        const diffMin = prayerTimeInMin - currentTimeInMin;
                        const hours = Math.floor(diffMin / 60);
                        const minutes = diffMin % 60;
                        
                        let remaining = '';
                        if (hours > 0) {
                            remaining = `${hours} ساعة و ${minutes} دقيقة`;
                        } else {
                            remaining = `${minutes} دقيقة`;
                        }
                        
                        this.state.timeRemaining = remaining;
                        return {name: prayer, time: time};
                    }
                }
                
                // Next day's Fajr
                const fajrTimeInMin = timeToMinutes(this.state.prayerTimes.fajr);
                const diffMin = (24 * 60) - currentTimeInMin + fajrTimeInMin;
                const hours = Math.floor(diffMin / 60);
                const minutes = diffMin % 60;
                this.state.timeRemaining = `${hours} ساعة و ${minutes} دقيقة`;
                
                return {name: 'fajr', time: this.state.prayerTimes.fajr};
            },
            
            
            
            handleLogoUpload(event) {
                const file = event.target.files[0];
                if (file && file.type.startsWith('image/')) {
                    const reader = new FileReader();
                    reader.onload = (e) => {
                        this.state.customLogo = e.target.result;
                        localStorage.setItem('customLogo', e.target.result);
                        alert('✅ Logo uploaded successfully!');
                        this.render();
                    };
                    reader.readAsDataURL(file);
                } else {
                    alert('Please upload a valid image file (PNG, JPG, etc.)');
                }
            },
            
            shareApp() {
                const shareData = {
                    title: 'أبو داود - Islamic Prayer App',
                    text: 'Check out this beautiful Islamic prayer times app for Saida, Lebanon! Features: Daily prayer times, Adhan audio, Athkar, and more. Made by Abu Dawod.',
                    url: window.location.href
                };
                
                if (navigator.share) {
                    navigator.share(shareData)
                        .then(() => {
                            document.getElementById('shareStatus').textContent = '✅ Shared successfully!';
                            setTimeout(() => {
                                document.getElementById('shareStatus').textContent = '';
                            }, 3000);
                        })
                        .catch((error) => {
                            if (error.name !== 'AbortError') {
                                this.copyLink();
                            }
                        });
                } else {
                    this.copyLink();
                }
            },
            
            copyLink() {
                const link = window.location.href;
                
                if (navigator.clipboard && navigator.clipboard.writeText) {
                    navigator.clipboard.writeText(link)
                        .then(() => {
                            document.getElementById('shareStatus').textContent = '✅ Link copied to clipboard!';
                            setTimeout(() => {
                                document.getElementById('shareStatus').textContent = '';
                            }, 3000);
                        })
                        .catch(() => {
                            this.fallbackCopyLink(link);
                        });
                } else {
                    this.fallbackCopyLink(link);
                }
            },
            
            fallbackCopyLink(link) {
                const textArea = document.createElement('textarea');
                textArea.value = link;
                textArea.style.position = 'fixed';
                textArea.style.left = '-999999px';
                document.body.appendChild(textArea);
                textArea.focus();
                textArea.select();
                
                try {
                    document.execCommand('copy');
                    document.getElementById('shareStatus').textContent = '✅ Link copied! Share it with WhatsApp, Telegram, etc.';
                    setTimeout(() => {
                        document.getElementById('shareStatus').textContent = '';
                    }, 5000);
                } catch (err) {
                    document.getElementById('shareStatus').textContent = '⚠️ Please manually copy: ' + link;
                }
                
                document.body.removeChild(textArea);
            },
            
            handleAudioUpload(event) {
                const file = event.target.files[0];
                if (file && file.type.startsWith('audio/')) {
                    const audioUrl = URL.createObjectURL(file);
                    this.state.uploadedAudio = audioUrl;
                    alert('✅ Audio file uploaded successfully! Click Play to listen.');
                    this.render();
                } else {
                    alert('Please upload a valid audio file (MP3, WAV, etc.)');
                }
            },
            
            playAdhan() {
                if(this.state.isPlayingAdhan && this.state.currentAudio) {
                    this.state.currentAudio.pause();
                    this.state.currentAudio.currentTime = 0;
                    this.state.isPlayingAdhan = false;
                    this.state.currentAudio = null;
                    this.render();
                    return;
                }
                
                // Use uploaded audio, or selected muadhun URL
                let audioSource = this.state.uploadedAudio || this.muadhuns[this.state.selectedMuadhun].url;
                
                if (!audioSource) {
                    alert('Please upload an MP3 file first, or make sure the MP3 files are in the same folder as this HTML file.');
                    return;
                }
                
                const audio = new Audio(audioSource);
                
                audio.addEventListener('error', (e) => {
                    console.error('Audio error:', e);
                    alert('⚠️ Cannot load audio file.\n\n✅ INSTRUCTIONS:\n1. Save this HTML file in your Downloads folder\n2. Make sure your MP3 files are in the same folder\n3. Or use the "Upload Audio" button to select any MP3 file');
                    this.state.isPlayingAdhan = false;
                    this.render();
                });
                
                audio.play().then(() => {
                    console.log('✅ Audio playing!');
                    this.state.isPlayingAdhan = true;
                    this.state.currentAudio = audio;
                    this.render();
                }).catch((err) => {
                    console.error('Play error:', err);
                    if (err.name === 'NotAllowedError') {
                        alert('Browser blocked audio! Click anywhere on the page first, then try again.');
                    } else {
                        alert('⚠️ Cannot play audio.\n\n✅ SOLUTION: Use the "Upload Audio" button to select your MP3 file manually.');
                    }
                    this.state.isPlayingAdhan = false;
                    this.render();
                });
                
                audio.onended = () => {
                    this.state.isPlayingAdhan = false;
                    this.state.currentAudio = null;
                    this.render();
                };
            },
            
            async toggleNotifications() {
                if(!this.state.notificationsEnabled) {
                    if('Notification' in window) {
                        const perm = await Notification.requestPermission();
                        if(perm === 'granted') {
                            this.state.notificationsEnabled = true;
                            localStorage.setItem('notificationsEnabled', 'true');
                        }
                    }
                } else {
                    this.state.notificationsEnabled = false;
                    localStorage.setItem('notificationsEnabled', 'false');
                }
                this.render();
            },
            
            icon(name) {
                const icons = {
                    moon: '<path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path>',
                    bell: '<path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"></path><path d="M13.73 21a2 2 0 0 1-3.46 0"></path>',
                    clock: '<circle cx="12" cy="12" r="10"></circle><polyline points="12 6 12 12 16 14"></polyline>',
                    sun: '<circle cx="12" cy="12" r="5"></circle><line x1="12" y1="1" x2="12" y2="3"></line><line x1="12" y1="21" x2="12" y2="23"></line><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line><line x1="1" y1="12" x2="3" y2="12"></line><line x1="21" y1="12" x2="23" y2="12"></line><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line>',
                    book: '<path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20"></path><path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z"></path>',
                    heart: '<path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"></path>',
                    wifi: '<path d="M5 12.55a11 11 0 0 1 14.08 0"></path><path d="M1.42 9a16 16 0 0 1 21.16 0"></path><path d="M8.53 16.11a6 6 0 0 1 6.95 0"></path><line x1="12" y1="20" x2="12.01" y2="20"></line>',
                    wifioff: '<line x1="1" y1="1" x2="23" y2="23"></line><path d="M16.72 11.06A10.94 10.94 0 0 1 19 12.55"></path><path d="M5 12.55a10.94 10.94 0 0 1 5.17-2.39"></path><path d="M10.71 5.05A16 16 0 0 1 22.58 9"></path><path d="M1.42 9a15.91 15.91 0 0 1 4.7-2.88"></path><path d="M8.53 16.11a6 6 0 0 1 6.95 0"></path><line x1="12" y1="20" x2="12.01" y2="20"></line>',
                    refresh: '<polyline points="23 4 23 10 17 10"></polyline><polyline points="1 20 1 14 7 14"></polyline><path d="M3.51 9a9 9 0 0 1 14.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0 0 20.49 15"></path>',
                    pin: '<path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z"></path><circle cx="12" cy="10" r="3"></circle>'
                };
                return `<svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">${icons[name]}</svg>`;
            },
            
            
            translatePrayerName(prayer) {
                const names = {
                    fajr: 'الفجر',
                    sunrise: 'الشروق',
                    dhuhr: 'الظهر',
                    asr: 'العصر',
                    maghrib: 'المغرب',
                    isha: 'العشاء'
                };
                return names[prayer] || prayer;
            },
            
            render() {
                const s = this.state;
                const next = this.getNextPrayer();
                
                document.getElementById('app').innerHTML = `
                    <div class="min-h-screen">
                        <!-- App Header with Logo -->
                        <div class="bg-gradient-to-r from-green-600 to-blue-600 text-white p-6 shadow-lg">
                            <!-- App Logo -->
                            <div class="app-logo arabic-text">أد</div>
                            
                            <div class="flex items-center justify-between flex-wrap gap-4">
                                <div class="flex items-center space-x-3">
                                    <div>
                                        <h1 class="text-2xl font-bold arabic-text">أبو داود</h1>
                                        <p class="text-sm text-green-100">Prayer Times - Saida, Lebanon</p>
                                    </div>
                                </div>
                                <div class="text-right">
                                    <div class="text-2xl font-bold">${s.currentTime.toLocaleTimeString('en-US', {hour: 'numeric', minute: '2-digit', hour12: true})}</div>
                                    <div class="text-sm opacity-90">${s.currentTime.toLocaleDateString()}</div>
                                    ${s.currentHijriDate ? `<div class="text-xs opacity-75 mt-1 arabic-text">${s.currentHijriDate}</div>` : ''}
                                </div>
                            </div>
                            
                            <!-- Custom Logo Section -->
                            ${s.customLogo ? `
                                <div class="mt-6 text-center">
                                    <img src="${s.customLogo}" class="custom-logo-img mx-auto" style="max-height: 120px; border-radius: 15px; box-shadow: 0 5px 15px rgba(0,0,0,0.2);" alt="App Logo">
                                </div>
                            ` : ''}
                        </div>

                        <!-- Tabs -->
                        <div class="bg-white shadow-sm">
                            <div class="flex overflow-x-auto" dir="rtl">
                                ${[
                                    {id: 'prayer', label: 'الصلاة'},
                                    {id: 'adhan', label: 'الأذان'},
                                    {id: 'athkar', label: 'الأذكار'},
                                    {id: 'tawba', label: 'التوبة'},
                                    {id: 'profile', label: '👤 الملف'}
                                ].map(tab => `
                                    <button onclick="app.state.activeTab='${tab.id}'; app.render()" 
                                        class="tab-button flex-1 flex items-center justify-center space-x-2 py-4 px-6 border-b-2 transition-colors whitespace-nowrap ${
                                            s.activeTab === tab.id ? 'border-green-500 text-green-600 bg-green-50' : 'border-transparent text-gray-500 hover:text-gray-700'
                                        }">
                                        <span class="font-medium arabic-text">${tab.label}</span>
                                    </button>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Made by Abu Dawod Banner -->
                        <div class="p-6 pb-4">
                            <div class="bg-gradient-to-r from-green-600 to-blue-600 text-white p-4 rounded-xl shadow-lg text-center">
                                <p class="text-lg font-semibold arabic-text">صُنع بـ ❤️ من قبل أبو داود</p>
                                <p class="text-sm opacity-90">May Allah accept this work and make it beneficial for all Muslims</p>
                            </div>
                        </div>

                        <!-- Content -->
                        <div class="p-6 pt-0">
                            ${s.activeTab === 'prayer' ? this.renderPrayer(next) : ''}
                            ${s.activeTab === 'adhan' ? this.renderAdhan() : ''}
                            ${s.activeTab === 'athkar' ? this.renderAthkar() : ''}
                            ${s.activeTab === 'tawba' ? this.renderTawba() : ''}
                            ${s.activeTab === 'profile' ? this.renderProfile() : ''}
                        </div>
                    </div>
                `;
            },
            
            renderProfile() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <!-- Developer Profile Header -->
                        <div class="bg-gradient-to-r from-green-600 via-emerald-600 to-blue-600 text-white p-8 rounded-xl shadow-2xl">
                            <div class="text-center">
                                <!-- Profile Picture Upload Area -->
                                <div class="mb-6">
                                    <div class="w-32 h-32 mx-auto rounded-full bg-white bg-opacity-20 flex items-center justify-center cursor-pointer hover:bg-opacity-30 transition-all overflow-hidden border-4 border-white shadow-lg" onclick="document.getElementById('logoUpload').click()">
                                        ${s.customLogo ? `
                                            <img src="${s.customLogo}" class="w-full h-full object-cover" alt="Profile">
                                        ` : `
                                            <svg class="w-16 h-16 text-white opacity-75" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"></path>
                                            </svg>
                                        `}
                                    </div>
                                    <input 
                                        type="file" 
                                        id="logoUpload"
                                        accept="image/*" 
                                        onchange="app.handleLogoUpload(event)"
                                        style="display: none;"
                                    />
                                    <p class="text-sm mt-2 opacity-75">Click to upload profile photo</p>
                                </div>
                                
                                <h1 class="text-4xl font-bold mb-2 arabic-text">أبو داود</h1>
                                <p class="text-xl mb-2">Abu Dawod</p>
                                <p class="text-lg opacity-90">Islamic App Developer</p>
                                <div class="flex items-center justify-center gap-2 mt-4">
                                    <span class="px-4 py-1 bg-white bg-opacity-20 rounded-full text-sm">Saida, Lebanon 🇱🇧</span>
                                </div>
                            </div>
                        </div>

                        <!-- About This Project -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">📱 About This Application</h2>
                            <div class="space-y-4 text-gray-700">
                                <p class="leading-relaxed">
                                    <strong class="text-green-600">Islamic Prayer Times & Athkar App</strong> is a comprehensive web application designed to help Muslims stay connected with their daily prayers and remembrance of Allah.
                                </p>
                                <p class="leading-relaxed">
                                    This app provides accurate prayer times for Saida, Lebanon, beautiful Adhan audio, daily Athkar, and reminders for Tawba (repentance).
                                </p>
                            </div>
                        </div>

                        <!-- Features -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">✨ Key Features</h2>
                            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                                ${[
                                    { icon: '🕌', title: 'Daily Prayer Times', desc: 'Accurate times for Saida updated automatically' },
                                    { icon: '🔔', title: 'Prayer Notifications', desc: 'Get notified when it\'s time to pray' },
                                    { icon: '🎵', title: 'Beautiful Adhan', desc: 'Multiple Adhan styles with audio playback' },
                                    { icon: '📿', title: 'Daily Athkar', desc: 'Morning, evening, and general remembrances' },
                                    { icon: '💜', title: 'Tawba Reminders', desc: 'Encouragement for repentance and forgiveness' },
                                    { icon: '🌙', title: 'Hijri Calendar', desc: 'Display Islamic date alongside Gregorian' },
                                    { icon: '📱', title: 'Installable App', desc: 'Works like a native mobile application' },
                                    { icon: '🔄', title: 'Auto-Updates', desc: 'Prayer times update daily automatically' }
                                ].map(f => `
                                    <div class="flex items-start space-x-3 p-3 bg-green-50 rounded-lg">
                                        <span class="text-2xl">${f.icon}</span>
                                        <div>
                                            <h3 class="font-semibold text-gray-800">${f.title}</h3>
                                            <p class="text-sm text-gray-600">${f.desc}</p>
                                        </div>
                                    </div>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Technologies Used -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">🛠️ Technologies Used</h2>
                            <div class="flex flex-wrap gap-3">
                                ${['HTML5', 'CSS3', 'JavaScript', 'Tailwind CSS', 'AlAdhan API', 'Web Audio API', 'Local Storage', 'PWA'].map(tech => `
                                    <span class="px-4 py-2 bg-gradient-to-r from-green-100 to-blue-100 text-gray-800 rounded-full text-sm font-semibold border border-green-200">${tech}</span>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Statistics -->
                        <div class="bg-gradient-to-r from-purple-50 to-pink-50 p-6 rounded-xl shadow-lg border-2 border-purple-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">📊 App Statistics</h2>
                            <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
                                ${[
                                    { num: '5', label: 'Daily Prayers' },
                                    { num: '10+', label: 'Athkar Duas' },
                                    { num: '4', label: 'Adhan Styles' },
                                    { num: '100%', label: 'Free & Open' }
                                ].map(stat => `
                                    <div class="text-center p-4 bg-white rounded-lg shadow-sm">
                                        <div class="text-3xl font-bold text-green-600">${stat.num}</div>
                                        <div class="text-sm text-gray-600 mt-1">${stat.label}</div>
                                    </div>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Contact & Share -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">🤝 Share This App</h2>
                            <p class="text-gray-700 mb-4">Help spread the benefit! Share this Islamic prayer app with your community:</p>
                            <div class="space-y-3">
                                <button onclick="app.shareApp()" 
                                    class="w-full px-6 py-3 bg-gradient-to-r from-green-500 to-blue-500 text-white rounded-lg font-semibold hover:from-green-600 hover:to-blue-600 transition-all shadow-md">
                                    📤 Share App
                                </button>
                                <button onclick="app.copyLink()" 
                                    class="w-full px-6 py-3 bg-gray-100 text-gray-800 rounded-lg font-semibold hover:bg-gray-200 transition-all">
                                    📋 Copy App Link
                                </button>
                                <div class="text-center text-sm text-gray-500 mt-2">
                                    <p id="shareStatus"></p>
                                </div>
                            </div>
                        </div>

                        <!-- Credits & License -->
                        <div class="bg-gradient-to-r from-green-600 to-blue-600 text-white p-6 rounded-xl shadow-lg text-center">
                            <h2 class="text-2xl font-bold mb-3 arabic-text">بارك الله فيك</h2>
                            <p class="text-lg mb-4">May Allah bless this work and make it a source of continuous reward (Sadaqah Jariyah)</p>
                            <div class="text-sm opacity-90">
                                <p>Developed with ❤️ by <strong class="arabic-text">أبو داود</strong></p>
                                <p class="mt-2">Version 1.0.0 | 2025</p>
                                <p class="mt-2">Free for all Muslims to use and benefit from</p>
                            </div>
                        </div>

                        ${s.customLogo ? `
                            <div class="bg-white p-4 rounded-lg shadow-sm border border-gray-200 text-center">
                                <button 
                                    onclick="app.state.customLogo = null; localStorage.removeItem('customLogo'); app.render();"
                                    class="px-6 py-2 bg-red-50 hover:bg-red-100 text-red-600 rounded-lg font-medium transition-colors">
                                    🗑️ Remove Profile Photo
                                </button>
                            </div>
                        ` : ''}
                    </div>
                `;
            },
            
            renderSettings() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-purple-500 to-indigo-500 text-white p-6 rounded-xl shadow-lg">
                            <h2 class="text-2xl font-bold mb-2">⚙️ Settings</h2>
                            <p class="text-lg opacity-90">Upload your program image</p>
                        </div>

                        <!-- Upload Photo Only -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-lg font-semibold mb-4 text-center">📸 Program Image</h3>
                            
                            <div class="custom-logo-container" onclick="document.getElementById('logoUpload').click()">
                                ${s.customLogo ? `
                                    <img src="${s.customLogo}" class="custom-logo-img" alt="Program Image">
                                ` : `
                                    <div class="text-center text-gray-400">
                                        <svg class="w-16 h-16 mx-auto mb-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"></path>
                                        </svg>
                                        <p class="text-sm font-semibold">Click to upload photo</p>
                                    </div>
                                `}
                            </div>
                            
                            <input 
                                type="file" 
                                id="logoUpload"
                                accept="image/*" 
                                onchange="app.handleLogoUpload(event)"
                                style="display: none;"
                            />
                            
                            ${s.customLogo ? `
                                <button 
                                    onclick="app.state.customLogo = null; localStorage.removeItem('customLogo'); app.render();"
                                    class="w-full mt-4 px-4 py-3 bg-red-50 hover:bg-red-100 text-red-600 rounded-lg font-semibold transition-colors">
                                    🗑️ Remove Photo
                                </button>
                            ` : ''}
                        </div>
                    </div>
                `;
            },
            
            renderPrayer(next) {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-green-500 to-blue-500 text-white p-6 rounded-xl shadow-lg transform transition-all hover:shadow-2xl">
                            <div class="text-center">
                                <h3 class="text-lg font-semibold opacity-90 mb-2">الصلاة القادمة</h3>
                                <h2 class="text-4xl font-bold mb-2 arabic-text">${this.translatePrayerName(next.name)}</h2>
                                <p class="text-2xl opacity-90 mb-3">${next.time}</p>
                                
                                <!-- Time Remaining -->
                                <div class="bg-white bg-opacity-20 rounded-lg p-4 mt-4">
                                    <p class="text-sm opacity-90 mb-1">الوقت المتبقي</p>
                                    <p class="text-3xl font-bold arabic-text">${s.timeRemaining}</p>
                                </div>
                            </div>
                        </div>

                        <div class="bg-white p-4 rounded-lg shadow-sm border border-gray-200">
                            <div class="flex items-center justify-between mb-3">
                                <div class="flex items-center space-x-3 space-x-reverse">
                                    ${this.icon('bell')}
                                    <span class="font-medium arabic-text">تنبيهات الصلاة</span>
                                </div>
                                <button onclick="app.toggleNotifications()" 
                                    class="relative inline-flex h-6 w-11 items-center rounded-full transition-colors ${s.notificationsEnabled ? 'bg-green-600' : 'bg-gray-200'}">
                                    <span class="inline-block h-4 w-4 transform rounded-full bg-white transition-transform ${s.notificationsEnabled ? 'translate-x-6' : 'translate-x-1'}"></span>
                                </button>
                            </div>
                            <div class="flex items-center justify-between text-sm pt-3 border-t border-gray-100">
                                <div class="flex items-center space-x-2 space-x-reverse">
                                    ${s.connectionStatus === 'connected' ? `${this.icon('wifi')}<span class="text-green-600 arabic-text">متصل بصيدا</span>` : 
                                      s.connectionStatus === 'connecting' ? `<div class="animate-spin">${this.icon('refresh')}</div><span class="text-blue-600 arabic-text">جاري الاتصال...</span>` :
                                      `${this.icon('wifioff')}<span class="text-red-600 arabic-text">غير متصل</span>`}
                                </div>
                                ${s.lastUpdated ? `<span class="text-gray-500 arabic-text">آخر تحديث: ${s.lastUpdated.toLocaleTimeString('ar-LB', {hour: 'numeric', minute: '2-digit', hour12: true})}</span>` : ''}
                            </div>
                            <button onclick="app.fetchPrayerTimes()" ${s.isLoading ? 'disabled' : ''}
                                class="w-full mt-3 px-4 py-2 bg-blue-50 hover:bg-blue-100 text-blue-600 rounded-lg font-medium transition-colors disabled:opacity-50 arabic-text">
                                ${s.isLoading ? '🔄 جاري التحديث...' : '🔄 تحديث أوقات الصلاة'}
                            </button>
                        </div>

                        <div class="bg-white rounded-lg shadow-sm border border-gray-200 overflow-hidden">
                            <h3 class="text-lg font-semibold p-4 border-b border-gray-200 bg-gray-50 arabic-text text-right">أوقات الصلاة اليوم</h3>
                            <div class="divide-y divide-gray-100">
                                ${Object.entries(s.prayerTimes).map(([p,t]) => `
                                    <div class="flex items-center justify-between p-4 hover:bg-gray-50 transition-colors" dir="rtl">
                                        <div class="flex items-center space-x-3 space-x-reverse">
                                            <div class="w-3 h-3 rounded-full ${p === next.name ? 'bg-green-500 animate-pulse' : 'bg-gray-300'}"></div>
                                            <span class="font-medium text-gray-700 arabic-text text-lg">${this.translatePrayerName(p)}</span>
                                        </div>
                                        <span class="text-lg font-semibold text-gray-900">${t}</span>
                                    </div>
                                `).join('')}
                            </div>
                        </div>
                    </div>
                `;
            },
            
            renderAdhan() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-amber-500 to-orange-500 text-white p-6 rounded-xl shadow-lg">
                            <div class="flex items-center space-x-3 mb-4">
                                ${this.icon('sun')}
                                <h2 class="text-2xl font-bold">Adhan Player</h2>
                            </div>
                            <p class="text-lg opacity-90">Listen to the beautiful call to prayer</p>
                        </div>

                        <!-- Instructions -->
                        <div class="bg-blue-50 p-4 rounded-lg border-2 border-blue-200">
                            <h3 class="font-bold text-blue-800 mb-2">📋 Instructions:</h3>
                            <ol class="text-sm text-blue-700 space-y-1 list-decimal list-inside">
                                <li>Save this HTML file in your <strong>Downloads folder</strong></li>
                                <li>Your MP3 files should be in the <strong>same folder</strong></li>
                                <li>Or click "Choose File" below to upload any MP3</li>
                            </ol>
                        </div>

                        <!-- Upload Audio -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-lg font-semibold mb-4">📁 Upload Your Adhan MP3</h3>
                            <input
                                type="file"
                                accept="audio/*"
                                onchange="app.handleAudioUpload(event)"
                                class="block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-green-50 file:text-green-700 hover:file:bg-green-100 cursor-pointer"
                            />
                            ${s.uploadedAudio ? '<p class="text-green-600 text-sm mt-2">✅ Audio uploaded! Ready to play.</p>' : ''}
                        </div>

                        <!-- Choose Adhan -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-lg font-semibold mb-4">Choose Adhan Source</h3>
                            <div class="space-y-3">
                                ${Object.entries(this.muadhuns).map(([k,m]) => `
                                    <div onclick="app.state.selectedMuadhun='${k}'; app.render()" 
                                        class="p-4 rounded-lg border-2 cursor-pointer transition-all transform hover:scale-102 ${
                                            s.selectedMuadhun === k ? 'border-amber-500 bg-amber-50 shadow-md' : 'border-gray-200 hover:border-gray-300'
                                        }">
                                        <div class="flex items-center justify-between">
                                            <div>
                                                <h4 class="font-semibold text-gray-800">${m.name}</h4>
                                                <p class="text-sm text-gray-600">${m.desc}</p>
                                            </div>
                                            <div class="w-4 h-4 rounded-full border-2 ${s.selectedMuadhun === k ? 'border-amber-500 bg-amber-500' : 'border-gray-300'}"></div>
                                        </div>
                                    </div>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Play Button -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <div class="text-center space-y-4">
                                <h3 class="text-lg font-semibold">${this.muadhuns[s.selectedMuadhun].name}</h3>
                                <button onclick="app.playAdhan()" 
                                    class="px-8 py-4 rounded-full text-white font-semibold text-lg transition-all transform hover:scale-105 shadow-lg ${
                                        s.isPlayingAdhan ? 'bg-red-500 hover:bg-red-600 animate-pulse' : 'bg-gradient-to-r from-green-500 to-blue-500 hover:from-green-600 hover:to-blue-600'
                                    }">
                                    ${s.isPlayingAdhan ? '⏸️ Stop Adhan' : '▶️ Play Adhan'}
                                </button>
                                ${s.isPlayingAdhan ? `
                                    <div class="bg-green-50 p-4 rounded-lg border border-green-200 animate-pulse">
                                        <p class="text-green-700 font-medium">🎵 Playing Adhan...</p>
                                        <p class="text-sm text-gray-600 mt-1">${s.uploadedAudio ? 'Your uploaded audio' : this.muadhuns[s.selectedMuadhun].name}</p>
                                    </div>
                                ` : ''}
                                <p class="text-sm text-gray-500 mt-2">🔊 Make sure your volume is on</p>
                            </div>
                        </div>
                    </div>
                `;
            },
            
            renderAthkar() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-white p-4 rounded-lg shadow-sm border border-gray-200">
                            <div class="flex space-x-2 overflow-x-auto">
                                ${Object.keys(this.athkar).map(cat => `
                                    <button onclick="app.state.selectedAthkar='${cat}'; app.render()" 
                                        class="px-4 py-2 rounded-lg font-medium capitalize transition-all whitespace-nowrap ${
                                            s.selectedAthkar === cat ? 'bg-green-500 text-white shadow-md' : 'bg-gray-100 text-gray-700 hover:bg-gray-200'
                                        }">
                                        ${cat}
                                    </button>
                                `).join('')}
                            </div>
                        </div>

                        ${this.athkar[s.selectedAthkar].map(d => `
                            <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200 transform transition-all hover:shadow-lg">
                                <div class="text-right mb-4">
                                    <p class="text-2xl text-green-700 leading-relaxed">${d.ar}</p>
                                </div>
                                <div class="space-y-2">
                                    <p class="text-gray-600 italic text-sm">${d.tr}</p>
                                    <p class="text-gray-800 font-medium">${d.en}</p>
                                </div>
                            </div>
                        `).join('')}
                    </div>
                `;
            },
            
            renderTawba() {
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-purple-500 to-pink-500 text-white p-6 rounded-xl shadow-lg">
                            <div class="flex items-center space-x-3 space-x-reverse mb-4" dir="rtl">
                                ${this.icon('heart')}
                                <h2 class="text-2xl font-bold arabic-text">التوبة - الاستغفار</h2>
                            </div>
                            <p class="text-lg opacity-90 arabic-text text-right leading-relaxed">
                                "وَمَن تَابَ بَعْدَ ظُلْمِهِ وَأَصْلَحَ فَإِنَّ اللَّهَ يَتُوبُ عَلَيْهِ ۗ إِنَّ اللَّهَ غَفُورٌ رَّحِيمٌ" - سورة المائدة 39
                            </p>
                        </div>

                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-xl font-semibold mb-4 text-center arabic-text">الاستغفار اليومي</h3>
                            <div class="text-center">
                                <div class="text-4xl font-bold text-green-600 mb-4 arabic-text leading-relaxed">
                                    أَسْتَغْفِرُ اللَّهَ الْعَظِيمَ
                                </div>
                                <p class="text-gray-600 mb-2 italic">Astaghfiru Allah al-Adheem</p>
                                <p class="text-sm text-gray-500 mb-4 arabic-text">أستغفر الله العظيم وأتوب إليه</p>
                            </div>
                        </div>

                        <div class="space-y-4">
                            <h3 class="text-xl font-semibold text-gray-700 arabic-text text-right">أدعية التوبة والاستغفار</h3>
                            
                            <div class="bg-gradient-to-r from-blue-50 to-purple-50 p-6 rounded-lg border-l-4 border-purple-400">
                                <p class="text-2xl text-gray-800 arabic-text text-right leading-loose mb-3">
                                    اللَّهُمَّ إِنَّكَ عَفُوٌّ كَرِيمٌ تُحِبُّ الْعَفْوَ فَاعْفُ عَنَّا
                                </p>
                                <p class="text-sm text-gray-600 italic">Allahumma innaka 'afuwwun karimun tuhibbul-'afwa fa'fu 'anna</p>
                                <p class="text-sm text-gray-700 mt-2 arabic-text text-right">
                                    اللهم إنك عفو كريم تحب العفو فاعف عنا
                                </p>
                            </div>

                            <div class="bg-gradient-to-r from-green-50 to-blue-50 p-6 rounded-lg border-l-4 border-green-400">
                                <p class="text-2xl text-gray-800 arabic-text text-right leading-loose mb-3">
                                    رَبَّنَا آتِنَا فِي الدُّنْيَا حَسَنَةً وَفِي الْآخِرَةِ حَسَنَةً وَقِنَا عَذَابَ النَّارِ
                                </p>
                                <p class="text-sm text-gray-600 italic">Rabbana atina fid-dunya hasanatan wa fil-akhirati hasanatan wa qina 'adhaban-nar</p>
                                <p class="text-sm text-gray-700 mt-2 arabic-text text-right">
                                    اللهم آتنا في الدنيا حسنة وفي الآخرة حسنة وقنا عذاب النار
                                </p>
                            </div>

                            <div class="bg-gradient-to-r from-purple-50 to-pink-50 p-6 rounded-lg border-l-4 border-pink-400">
                                <p class="text-2xl text-gray-800 arabic-text text-right leading-loose mb-3">
                                    اللَّهُمَّ آتِ نَفْسِي تَقْوَاهَا وَزَكِّهَا أَنتَ خَيْرُ مَن زَكَّاهَا أَنتَ وَلِيُّهَا وَمَوْلَاهَا
                                </p>
                                <p class="text-sm text-gray-600 italic">Allahumma ati nafsi taqwaha wa zakkiha anta khayru man zakkaha anta waliyyuha wa mawlaha</p>
                                <p class="text-sm text-gray-700 mt-2 arabic-text text-right">
                                    اللهم آت نفسي تقواها وزكها أنت خير من زكاها أنت وليها ومولاها
                                </p>
                            </div>
                        </div>

                        <div class="space-y-4">
                            <h3 class="text-xl font-semibold text-gray-700 arabic-text text-right">تذكيرات يومية</h3>
                            ${this.tawbaReminders.map(r => `
                                <div class="bg-gradient-to-r from-blue-50 to-purple-50 p-4 rounded-lg border-r-4 border-purple-400 transform transition-all hover:shadow-md" dir="rtl">
                                    <p class="text-gray-700 arabic-text text-right">${r}</p>
                                </div>
                            `).join('')}
                        </div>

                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-xl font-semibold mb-4 arabic-text text-right">دعاء الاستغفار</h3>
                            <div class="border-r-4 border-green-400 pr-4" dir="rtl">
                                <p class="text-2xl text-green-700 mb-3 text-right leading-loose arabic-text">
                                    رَبَّنَا ظَلَمْنَا أَنفُسَنَا وَإِن لَّمْ تَغْفِرْ لَنَا وَتَرْحَمْنَا لَنَكُونَنَّ مِنَ الْخَاسِرِينَ
                                </p>
                                <p class="text-sm text-gray-600 italic mb-1">
                                    Rabbana zalamna anfusana wa in lam taghfir lana wa tarhamna lanakoonanna minal-khasireen
                                </p>
                                <p class="text-sm text-gray-700 arabic-text text-right">
                                    "رَبَّنَا ظَلَمْنَا أَنفُسَنَا وَإِن لَّمْ تَغْفِرْ لَنَا وَتَرْحَمْنَا لَنَكُونَنَّ مِنَ الْخَاسِرِينَ"
                                </p>
                            </div>
                        </div>
                    </div>
                `;
            },
            
            init() {
                this.fetchPrayerTimes();
                setInterval(() => {
                    this.state.currentTime = new Date();
                    this.render();
                }, 1000);
                setInterval(() => this.fetchPrayerTimes(), 21600000);
                setInterval(() => {
                    const now = new Date();
                    if(now.getHours() === 0 && now.getMinutes() === 0) {
                        this.fetchPrayerTimes();
                    }
                }, 60000);
                this.render();
            }
        };
        
        window.app = app;
        app.init();
    </script>
</body>
</html><!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <meta name="author" content="Abu Dawod">
    <meta name="description" content="Islamic Prayer Times App for Saida, Lebanon - Made by Abu Dawod">
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="apple-mobile-web-app-title" content="أبو داود">
    <meta name="application-name" content="أبو داود">
    <meta name="theme-color" content="#16a34a">
    
    <!-- PWA Manifest -->
    <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoi2KPYqNmIINiv2KfZiNivIiwic2hvcnRfbmFtZSI6Itin2KjZiCDYr9in2YjYryIsInN0YXJ0X3VybCI6Ii4vaW5kZXguaHRtbCIsImRpc3BsYXkiOiJzdGFuZGFsb25lIiwiYmFja2dyb3VuZF9jb2xvciI6IiNmZmZmZmYiLCJ0aGVtZV9jb2xvciI6IiMxNmEzNGEiLCJpY29ucyI6W3sic3JjIjoiZGF0YTppbWFnZS9zdmcreG1sO2Jhc2U2NCxQSE4yWnlCNGJXeHVjejBpYUhSMGNEb3ZMM2QzZHk1M015NXZjbWN2TWpBd01DOXpkbWNpSUhacFpYZENiM2c5SWpBZ01DQXlOQ0F5TkNJK1BHTnBjbU5zWlNCamVEMGlNVElpSUdONVBTSXhNaUlnY2owSU5pSWdabWxzYkQwaUl6RTJZVE0wWVNJdlBqd3ZjM1puUGc9PSIsInNpemVzIjoiNTEyeDUxMiIsInR5cGUiOiJpbWFnZS9zdmcreG1sIn1dfQ==">
    
    <!-- Favicon / App Icon -->
    <link rel="icon" type="image/svg+xml" href="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI+PGNpcmNsZSBjeD0iMTIiIGN5PSIxMiIgcj0iNiIgZmlsbD0iIzE2YTM0YSIvPjwvc3ZnPg==">
    
    <title>أبو داود - Prayer Times</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .animate-pulse { animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite; }
        @keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: .5; } }
        .animate-spin { animation: spin 1s linear infinite; }
        @keyframes spin { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }
        .transition-all { transition: all 0.3s ease; }
        .transform:hover { transform: scale(1.02); }
        .tab-button { transition: all 0.3s ease; }
        .tab-button:hover { background-color: rgba(0,0,0,0.05); }
    </style>
</head>
<body class="bg-gradient-to-br from-green-50 to-blue-50 min-h-screen">
    <div id="app"></div>

    <script>
        const app = {
            state: {
                currentTime: new Date(),
                activeTab: 'prayer',
                notificationsEnabled: localStorage.getItem('notificationsEnabled') === 'true',
                selectedAthkar: 'morning',
                selectedMuadhun: 'local1',
                isPlayingAdhan: false,
                currentAudio: null,
                uploadedAudio: null,
                prayerTimes: {
                    fajr: '05:30',
                    sunrise: '06:45',
                    dhuhr: '12:15',
                    asr: '15:45',
                    maghrib: '18:20',
                    isha: '19:45'
                },
                isLoading: false,
                lastUpdated: null,
                connectionStatus: 'loading',
                currentHijriDate: '',
                customLogo: localStorage.getItem('customLogo') || null
            },
            
            SAIDA: { latitude: 33.5631, longitude: 35.3708, city: 'Saida', country: 'Lebanon' },
            
            muadhuns: {
                local1: { name: 'Your Adhan 1 (haii3alslah936584)', desc: 'Local MP3 file from Downloads', url: './haii3alslah936584 (1).mp3' },
                local2: { name: 'Your Adhan 2 (haii3alslah2336584)', desc: 'Local MP3 file from Downloads', url: './haii3alslah2336584.mp3' },
                upload: { name: 'Upload Your Own', desc: 'Click to upload any MP3 file', url: null }
            },
            
            athkar: {
                morning: [
                    { ar: 'اللَّهُمَّ أَعِنِّي عَلَى ذِكْرِكَ وَشُكْرِكَ وَحُسْنِ عِبَادَتِكَ', tr: 'Allahumma a\'inni ala dhikrika wa shukrika wa husni ibadatika', en: 'O Allah, help me to remember You, to thank You, and to worship You in the best manner.' },
                    { ar: 'أَصْبَحْنَا وَأَصْبَحَ الْمُلْكُ لِلَّهِ', tr: 'Asbahna wa asbaha al-mulku lillahi', en: 'We have entered the morning and the whole kingdom belongs to Allah.' },
                    { ar: 'اللَّهُمَّ بِكَ أَصْبَحْنَا وَبِكَ أَمْسَيْنَا وَبِكَ نَحْيَا وَبِكَ نَمُوتُ وَإِلَيْكَ النُّشُورُ', tr: 'Allahumma bika asbahna wa bika amsayna wa bika nahya wa bika namutu wa ilayka an-nushur', en: 'O Allah, by You we enter the morning and by You we enter the evening, by You we live and by You we die, and to You is the resurrection.' }
                ],
                evening: [
                    { ar: 'أَمْسَيْنَا وَأَمْسَى الْمُلْكُ لِلَّهِ', tr: 'Amsayna wa amsa al-mulku lillahi', en: 'We have entered the evening and the whole kingdom belongs to Allah.' },
                    { ar: 'اللَّهُمَّ إِنِّي أَعُوذُ بِكَ مِنْ عَذَابِ جَهَنَّمَ وَمِنْ عَذَابِ الْقَبْرِ', tr: 'Allahumma inni a\'udhu bika min adhabi jahannama wa min adhabi al-qabr', en: 'O Allah, I seek refuge in You from the punishment of Hell and from the punishment of the grave.' }
                ],
                general: [
                    { ar: 'سُبْحَانَ اللَّهِ وَبِحَمْدِهِ', tr: 'Subhan Allahi wa bihamdihi', en: 'Glory be to Allah and praise be to Him.' },
                    { ar: 'لَا إِلَهَ إِلَّا اللَّهُ وَحْدَهُ لَا شَرِيكَ لَهُ', tr: 'La ilaha illa Allah wahdahu la sharika lahu', en: 'There is no god but Allah alone, with no partner.' },
                    { ar: 'أَسْتَغْفِرُ اللَّهَ الْعَظِيمَ', tr: 'Astaghfiru Allah al-adheem', en: 'I seek forgiveness from Allah, the Most Great.' }
                ]
            },
            
            tawbaReminders: [
                'Remember to seek Allah\'s forgiveness regularly - He is Most Forgiving, Most Merciful.',
                'Turn to Allah in repentance before it\'s too late. Every moment is a chance for a new beginning.',
                'Allah loves those who turn to Him in repentance and purify themselves.',
                'No sin is too great for Allah\'s forgiveness when you sincerely repent.',
                'Make Istighfar (seeking forgiveness) a daily habit - it brings peace to the heart.'
            ],
            
            async fetchPrayerTimes() {
                this.state.isLoading = true;
                this.state.connectionStatus = 'connecting';
                this.render();
                
                try {
                    const d = new Date();
                    const url = `https://api.aladhan.com/v1/timings/${d.getDate()}-${d.getMonth()+1}-${d.getFullYear()}?latitude=${this.SAIDA.latitude}&longitude=${this.SAIDA.longitude}&method=8`;
                    const res = await fetch(url);
                    const data = await res.json();
                    
                    // Convert 24hr to 12hr format
                    const convert24to12 = (time24) => {
                        const [hours, minutes] = time24.split(':');
                        const h = parseInt(hours);
                        const period = h >= 12 ? 'PM' : 'AM';
                        const h12 = h === 0 ? 12 : h > 12 ? h - 12 : h;
                        return `${h12}:${minutes} ${period}`;
                    };
                    
                    this.state.prayerTimes = {
                        fajr: convert24to12(data.data.timings.Fajr),
                        sunrise: convert24to12(data.data.timings.Sunrise),
                        dhuhr: convert24to12(data.data.timings.Dhuhr),
                        asr: convert24to12(data.data.timings.Asr),
                        maghrib: convert24to12(data.data.timings.Maghrib),
                        isha: convert24to12(data.data.timings.Isha)
                    };
                    
                    this.state.currentHijriDate = `${data.data.date.hijri.day} ${data.data.date.hijri.month.en} ${data.data.date.hijri.year}`;
                    this.state.connectionStatus = 'connected';
                    this.state.lastUpdated = new Date();
                } catch(e) {
                    console.error(e);
                    this.state.connectionStatus = 'offline';
                }
                
                this.state.isLoading = false;
                this.render();
            },
            
            getNextPrayer() {
                const now = new Date();
                const currentHour = now.getHours();
                const currentMin = now.getMinutes();
                const currentTimeInMin = currentHour * 60 + currentMin;
                
                // Convert 12hr format back to minutes for comparison
                const timeToMinutes = (time12) => {
                    const match = time12.match(/(\d+):(\d+)\s*(AM|PM)/i);
                    if (!match) return 0;
                    let hours = parseInt(match[1]);
                    const minutes = parseInt(match[2]);
                    const period = match[3].toUpperCase();
                    
                    if (period === 'PM' && hours !== 12) hours += 12;
                    if (period === 'AM' && hours === 12) hours = 0;
                    
                    return hours * 60 + minutes;
                };
                
                const prayers = [
                    ['fajr', this.state.prayerTimes.fajr],
                    ['dhuhr', this.state.prayerTimes.dhuhr],
                    ['asr', this.state.prayerTimes.asr],
                    ['maghrib', this.state.prayerTimes.maghrib],
                    ['isha', this.state.prayerTimes.isha]
                ];
                
                for(let [prayer, time] of prayers) {
                    const prayerTimeInMin = timeToMinutes(time);
                    if(prayerTimeInMin > currentTimeInMin) {
                        // Calculate time remaining
                        const diffMin = prayerTimeInMin - currentTimeInMin;
                        const hours = Math.floor(diffMin / 60);
                        const minutes = diffMin % 60;
                        
                        let remaining = '';
                        if (hours > 0) {
                            remaining = `${hours} ساعة و ${minutes} دقيقة`;
                        } else {
                            remaining = `${minutes} دقيقة`;
                        }
                        
                        this.state.timeRemaining = remaining;
                        return {name: prayer, time: time};
                    }
                }
                
                // Next day's Fajr
                const fajrTimeInMin = timeToMinutes(this.state.prayerTimes.fajr);
                const diffMin = (24 * 60) - currentTimeInMin + fajrTimeInMin;
                const hours = Math.floor(diffMin / 60);
                const minutes = diffMin % 60;
                this.state.timeRemaining = `${hours} ساعة و ${minutes} دقيقة`;
                
                return {name: 'fajr', time: this.state.prayerTimes.fajr};
            },
            
            
            
            handleLogoUpload(event) {
                const file = event.target.files[0];
                if (file && file.type.startsWith('image/')) {
                    const reader = new FileReader();
                    reader.onload = (e) => {
                        this.state.customLogo = e.target.result;
                        localStorage.setItem('customLogo', e.target.result);
                        alert('✅ Logo uploaded successfully!');
                        this.render();
                    };
                    reader.readAsDataURL(file);
                } else {
                    alert('Please upload a valid image file (PNG, JPG, etc.)');
                }
            },
            
            shareApp() {
                const shareData = {
                    title: 'أبو داود - Islamic Prayer App',
                    text: 'Check out this beautiful Islamic prayer times app for Saida, Lebanon! Features: Daily prayer times, Adhan audio, Athkar, and more. Made by Abu Dawod.',
                    url: window.location.href
                };
                
                if (navigator.share) {
                    navigator.share(shareData)
                        .then(() => {
                            document.getElementById('shareStatus').textContent = '✅ Shared successfully!';
                            setTimeout(() => {
                                document.getElementById('shareStatus').textContent = '';
                            }, 3000);
                        })
                        .catch((error) => {
                            if (error.name !== 'AbortError') {
                                this.copyLink();
                            }
                        });
                } else {
                    this.copyLink();
                }
            },
            
            copyLink() {
                const link = window.location.href;
                
                if (navigator.clipboard && navigator.clipboard.writeText) {
                    navigator.clipboard.writeText(link)
                        .then(() => {
                            document.getElementById('shareStatus').textContent = '✅ Link copied to clipboard!';
                            setTimeout(() => {
                                document.getElementById('shareStatus').textContent = '';
                            }, 3000);
                        })
                        .catch(() => {
                            this.fallbackCopyLink(link);
                        });
                } else {
                    this.fallbackCopyLink(link);
                }
            },
            
            fallbackCopyLink(link) {
                const textArea = document.createElement('textarea');
                textArea.value = link;
                textArea.style.position = 'fixed';
                textArea.style.left = '-999999px';
                document.body.appendChild(textArea);
                textArea.focus();
                textArea.select();
                
                try {
                    document.execCommand('copy');
                    document.getElementById('shareStatus').textContent = '✅ Link copied! Share it with WhatsApp, Telegram, etc.';
                    setTimeout(() => {
                        document.getElementById('shareStatus').textContent = '';
                    }, 5000);
                } catch (err) {
                    document.getElementById('shareStatus').textContent = '⚠️ Please manually copy: ' + link;
                }
                
                document.body.removeChild(textArea);
            },
            
            handleAudioUpload(event) {
                const file = event.target.files[0];
                if (file && file.type.startsWith('audio/')) {
                    const audioUrl = URL.createObjectURL(file);
                    this.state.uploadedAudio = audioUrl;
                    alert('✅ Audio file uploaded successfully! Click Play to listen.');
                    this.render();
                } else {
                    alert('Please upload a valid audio file (MP3, WAV, etc.)');
                }
            },
            
            playAdhan() {
                if(this.state.isPlayingAdhan && this.state.currentAudio) {
                    this.state.currentAudio.pause();
                    this.state.currentAudio.currentTime = 0;
                    this.state.isPlayingAdhan = false;
                    this.state.currentAudio = null;
                    this.render();
                    return;
                }
                
                // Use uploaded audio, or selected muadhun URL
                let audioSource = this.state.uploadedAudio || this.muadhuns[this.state.selectedMuadhun].url;
                
                if (!audioSource) {
                    alert('Please upload an MP3 file first, or make sure the MP3 files are in the same folder as this HTML file.');
                    return;
                }
                
                const audio = new Audio(audioSource);
                
                audio.addEventListener('error', (e) => {
                    console.error('Audio error:', e);
                    alert('⚠️ Cannot load audio file.\n\n✅ INSTRUCTIONS:\n1. Save this HTML file in your Downloads folder\n2. Make sure your MP3 files are in the same folder\n3. Or use the "Upload Audio" button to select any MP3 file');
                    this.state.isPlayingAdhan = false;
                    this.render();
                });
                
                audio.play().then(() => {
                    console.log('✅ Audio playing!');
                    this.state.isPlayingAdhan = true;
                    this.state.currentAudio = audio;
                    this.render();
                }).catch((err) => {
                    console.error('Play error:', err);
                    if (err.name === 'NotAllowedError') {
                        alert('Browser blocked audio! Click anywhere on the page first, then try again.');
                    } else {
                        alert('⚠️ Cannot play audio.\n\n✅ SOLUTION: Use the "Upload Audio" button to select your MP3 file manually.');
                    }
                    this.state.isPlayingAdhan = false;
                    this.render();
                });
                
                audio.onended = () => {
                    this.state.isPlayingAdhan = false;
                    this.state.currentAudio = null;
                    this.render();
                };
            },
            
            async toggleNotifications() {
                if(!this.state.notificationsEnabled) {
                    if('Notification' in window) {
                        const perm = await Notification.requestPermission();
                        if(perm === 'granted') {
                            this.state.notificationsEnabled = true;
                            localStorage.setItem('notificationsEnabled', 'true');
                        }
                    }
                } else {
                    this.state.notificationsEnabled = false;
                    localStorage.setItem('notificationsEnabled', 'false');
                }
                this.render();
            },
            
            icon(name) {
                const icons = {
                    moon: '<path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path>',
                    bell: '<path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"></path><path d="M13.73 21a2 2 0 0 1-3.46 0"></path>',
                    clock: '<circle cx="12" cy="12" r="10"></circle><polyline points="12 6 12 12 16 14"></polyline>',
                    sun: '<circle cx="12" cy="12" r="5"></circle><line x1="12" y1="1" x2="12" y2="3"></line><line x1="12" y1="21" x2="12" y2="23"></line><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line><line x1="1" y1="12" x2="3" y2="12"></line><line x1="21" y1="12" x2="23" y2="12"></line><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line>',
                    book: '<path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20"></path><path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z"></path>',
                    heart: '<path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"></path>',
                    wifi: '<path d="M5 12.55a11 11 0 0 1 14.08 0"></path><path d="M1.42 9a16 16 0 0 1 21.16 0"></path><path d="M8.53 16.11a6 6 0 0 1 6.95 0"></path><line x1="12" y1="20" x2="12.01" y2="20"></line>',
                    wifioff: '<line x1="1" y1="1" x2="23" y2="23"></line><path d="M16.72 11.06A10.94 10.94 0 0 1 19 12.55"></path><path d="M5 12.55a10.94 10.94 0 0 1 5.17-2.39"></path><path d="M10.71 5.05A16 16 0 0 1 22.58 9"></path><path d="M1.42 9a15.91 15.91 0 0 1 4.7-2.88"></path><path d="M8.53 16.11a6 6 0 0 1 6.95 0"></path><line x1="12" y1="20" x2="12.01" y2="20"></line>',
                    refresh: '<polyline points="23 4 23 10 17 10"></polyline><polyline points="1 20 1 14 7 14"></polyline><path d="M3.51 9a9 9 0 0 1 14.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0 0 20.49 15"></path>',
                    pin: '<path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z"></path><circle cx="12" cy="10" r="3"></circle>'
                };
                return `<svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">${icons[name]}</svg>`;
            },
            
            
            translatePrayerName(prayer) {
                const names = {
                    fajr: 'الفجر',
                    sunrise: 'الشروق',
                    dhuhr: 'الظهر',
                    asr: 'العصر',
                    maghrib: 'المغرب',
                    isha: 'العشاء'
                };
                return names[prayer] || prayer;
            },
            
            render() {
                const s = this.state;
                const next = this.getNextPrayer();
                
                document.getElementById('app').innerHTML = `
                    <div class="min-h-screen">
                        <!-- App Header with Logo -->
                        <div class="bg-gradient-to-r from-green-600 to-blue-600 text-white p-6 shadow-lg">
                            <!-- App Logo -->
                            <div class="app-logo arabic-text">أد</div>
                            
                            <div class="flex items-center justify-between flex-wrap gap-4">
                                <div class="flex items-center space-x-3">
                                    <div>
                                        <h1 class="text-2xl font-bold arabic-text">أبو داود</h1>
                                        <p class="text-sm text-green-100">Prayer Times - Saida, Lebanon</p>
                                    </div>
                                </div>
                                <div class="text-right">
                                    <div class="text-2xl font-bold">${s.currentTime.toLocaleTimeString('en-US', {hour: 'numeric', minute: '2-digit', hour12: true})}</div>
                                    <div class="text-sm opacity-90">${s.currentTime.toLocaleDateString()}</div>
                                    ${s.currentHijriDate ? `<div class="text-xs opacity-75 mt-1 arabic-text">${s.currentHijriDate}</div>` : ''}
                                </div>
                            </div>
                            
                            <!-- Custom Logo Section -->
                            ${s.customLogo ? `
                                <div class="mt-6 text-center">
                                    <img src="${s.customLogo}" class="custom-logo-img mx-auto" style="max-height: 120px; border-radius: 15px; box-shadow: 0 5px 15px rgba(0,0,0,0.2);" alt="App Logo">
                                </div>
                            ` : ''}
                        </div>

                        <!-- Tabs -->
                        <div class="bg-white shadow-sm">
                            <div class="flex overflow-x-auto" dir="rtl">
                                ${[
                                    {id: 'prayer', label: 'الصلاة'},
                                    {id: 'adhan', label: 'الأذان'},
                                    {id: 'athkar', label: 'الأذكار'},
                                    {id: 'tawba', label: 'التوبة'},
                                    {id: 'profile', label: '👤 الملف'}
                                ].map(tab => `
                                    <button onclick="app.state.activeTab='${tab.id}'; app.render()" 
                                        class="tab-button flex-1 flex items-center justify-center space-x-2 py-4 px-6 border-b-2 transition-colors whitespace-nowrap ${
                                            s.activeTab === tab.id ? 'border-green-500 text-green-600 bg-green-50' : 'border-transparent text-gray-500 hover:text-gray-700'
                                        }">
                                        <span class="font-medium arabic-text">${tab.label}</span>
                                    </button>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Made by Abu Dawod Banner -->
                        <div class="p-6 pb-4">
                            <div class="bg-gradient-to-r from-green-600 to-blue-600 text-white p-4 rounded-xl shadow-lg text-center">
                                <p class="text-lg font-semibold arabic-text">صُنع بـ ❤️ من قبل أبو داود</p>
                                <p class="text-sm opacity-90">May Allah accept this work and make it beneficial for all Muslims</p>
                            </div>
                        </div>

                        <!-- Content -->
                        <div class="p-6 pt-0">
                            ${s.activeTab === 'prayer' ? this.renderPrayer(next) : ''}
                            ${s.activeTab === 'adhan' ? this.renderAdhan() : ''}
                            ${s.activeTab === 'athkar' ? this.renderAthkar() : ''}
                            ${s.activeTab === 'tawba' ? this.renderTawba() : ''}
                            ${s.activeTab === 'profile' ? this.renderProfile() : ''}
                        </div>
                    </div>
                `;
            },
            
            renderProfile() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <!-- Developer Profile Header -->
                        <div class="bg-gradient-to-r from-green-600 via-emerald-600 to-blue-600 text-white p-8 rounded-xl shadow-2xl">
                            <div class="text-center">
                                <!-- Profile Picture Upload Area -->
                                <div class="mb-6">
                                    <div class="w-32 h-32 mx-auto rounded-full bg-white bg-opacity-20 flex items-center justify-center cursor-pointer hover:bg-opacity-30 transition-all overflow-hidden border-4 border-white shadow-lg" onclick="document.getElementById('logoUpload').click()">
                                        ${s.customLogo ? `
                                            <img src="${s.customLogo}" class="w-full h-full object-cover" alt="Profile">
                                        ` : `
                                            <svg class="w-16 h-16 text-white opacity-75" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"></path>
                                            </svg>
                                        `}
                                    </div>
                                    <input 
                                        type="file" 
                                        id="logoUpload"
                                        accept="image/*" 
                                        onchange="app.handleLogoUpload(event)"
                                        style="display: none;"
                                    />
                                    <p class="text-sm mt-2 opacity-75">Click to upload profile photo</p>
                                </div>
                                
                                <h1 class="text-4xl font-bold mb-2 arabic-text">أبو داود</h1>
                                <p class="text-xl mb-2">Abu Dawod</p>
                                <p class="text-lg opacity-90">Islamic App Developer</p>
                                <div class="flex items-center justify-center gap-2 mt-4">
                                    <span class="px-4 py-1 bg-white bg-opacity-20 rounded-full text-sm">Saida, Lebanon 🇱🇧</span>
                                </div>
                            </div>
                        </div>

                        <!-- About This Project -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">📱 About This Application</h2>
                            <div class="space-y-4 text-gray-700">
                                <p class="leading-relaxed">
                                    <strong class="text-green-600">Islamic Prayer Times & Athkar App</strong> is a comprehensive web application designed to help Muslims stay connected with their daily prayers and remembrance of Allah.
                                </p>
                                <p class="leading-relaxed">
                                    This app provides accurate prayer times for Saida, Lebanon, beautiful Adhan audio, daily Athkar, and reminders for Tawba (repentance).
                                </p>
                            </div>
                        </div>

                        <!-- Features -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">✨ Key Features</h2>
                            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                                ${[
                                    { icon: '🕌', title: 'Daily Prayer Times', desc: 'Accurate times for Saida updated automatically' },
                                    { icon: '🔔', title: 'Prayer Notifications', desc: 'Get notified when it\'s time to pray' },
                                    { icon: '🎵', title: 'Beautiful Adhan', desc: 'Multiple Adhan styles with audio playback' },
                                    { icon: '📿', title: 'Daily Athkar', desc: 'Morning, evening, and general remembrances' },
                                    { icon: '💜', title: 'Tawba Reminders', desc: 'Encouragement for repentance and forgiveness' },
                                    { icon: '🌙', title: 'Hijri Calendar', desc: 'Display Islamic date alongside Gregorian' },
                                    { icon: '📱', title: 'Installable App', desc: 'Works like a native mobile application' },
                                    { icon: '🔄', title: 'Auto-Updates', desc: 'Prayer times update daily automatically' }
                                ].map(f => `
                                    <div class="flex items-start space-x-3 p-3 bg-green-50 rounded-lg">
                                        <span class="text-2xl">${f.icon}</span>
                                        <div>
                                            <h3 class="font-semibold text-gray-800">${f.title}</h3>
                                            <p class="text-sm text-gray-600">${f.desc}</p>
                                        </div>
                                    </div>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Technologies Used -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">🛠️ Technologies Used</h2>
                            <div class="flex flex-wrap gap-3">
                                ${['HTML5', 'CSS3', 'JavaScript', 'Tailwind CSS', 'AlAdhan API', 'Web Audio API', 'Local Storage', 'PWA'].map(tech => `
                                    <span class="px-4 py-2 bg-gradient-to-r from-green-100 to-blue-100 text-gray-800 rounded-full text-sm font-semibold border border-green-200">${tech}</span>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Statistics -->
                        <div class="bg-gradient-to-r from-purple-50 to-pink-50 p-6 rounded-xl shadow-lg border-2 border-purple-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">📊 App Statistics</h2>
                            <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
                                ${[
                                    { num: '5', label: 'Daily Prayers' },
                                    { num: '10+', label: 'Athkar Duas' },
                                    { num: '4', label: 'Adhan Styles' },
                                    { num: '100%', label: 'Free & Open' }
                                ].map(stat => `
                                    <div class="text-center p-4 bg-white rounded-lg shadow-sm">
                                        <div class="text-3xl font-bold text-green-600">${stat.num}</div>
                                        <div class="text-sm text-gray-600 mt-1">${stat.label}</div>
                                    </div>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Contact & Share -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">🤝 Share This App</h2>
                            <p class="text-gray-700 mb-4">Help spread the benefit! Share this Islamic prayer app with your community:</p>
                            <div class="space-y-3">
                                <button onclick="app.shareApp()" 
                                    class="w-full px-6 py-3 bg-gradient-to-r from-green-500 to-blue-500 text-white rounded-lg font-semibold hover:from-green-600 hover:to-blue-600 transition-all shadow-md">
                                    📤 Share App
                                </button>
                                <button onclick="app.copyLink()" 
                                    class="w-full px-6 py-3 bg-gray-100 text-gray-800 rounded-lg font-semibold hover:bg-gray-200 transition-all">
                                    📋 Copy App Link
                                </button>
                                <div class="text-center text-sm text-gray-500 mt-2">
                                    <p id="shareStatus"></p>
                                </div>
                            </div>
                        </div>

                        <!-- Credits & License -->
                        <div class="bg-gradient-to-r from-green-600 to-blue-600 text-white p-6 rounded-xl shadow-lg text-center">
                            <h2 class="text-2xl font-bold mb-3 arabic-text">بارك الله فيك</h2>
                            <p class="text-lg mb-4">May Allah bless this work and make it a source of continuous reward (Sadaqah Jariyah)</p>
                            <div class="text-sm opacity-90">
                                <p>Developed with ❤️ by <strong class="arabic-text">أبو داود</strong></p>
                                <p class="mt-2">Version 1.0.0 | 2025</p>
                                <p class="mt-2">Free for all Muslims to use and benefit from</p>
                            </div>
                        </div>

                        ${s.customLogo ? `
                            <div class="bg-white p-4 rounded-lg shadow-sm border border-gray-200 text-center">
                                <button 
                                    onclick="app.state.customLogo = null; localStorage.removeItem('customLogo'); app.render();"
                                    class="px-6 py-2 bg-red-50 hover:bg-red-100 text-red-600 rounded-lg font-medium transition-colors">
                                    🗑️ Remove Profile Photo
                                </button>
                            </div>
                        ` : ''}
                    </div>
                `;
            },
            
            renderSettings() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-purple-500 to-indigo-500 text-white p-6 rounded-xl shadow-lg">
                            <h2 class="text-2xl font-bold mb-2">⚙️ Settings</h2>
                            <p class="text-lg opacity-90">Upload your program image</p>
                        </div>

                        <!-- Upload Photo Only -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-lg font-semibold mb-4 text-center">📸 Program Image</h3>
                            
                            <div class="custom-logo-container" onclick="document.getElementById('logoUpload').click()">
                                ${s.customLogo ? `
                                    <img src="${s.customLogo}" class="custom-logo-img" alt="Program Image">
                                ` : `
                                    <div class="text-center text-gray-400">
                                        <svg class="w-16 h-16 mx-auto mb-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"></path>
                                        </svg>
                                        <p class="text-sm font-semibold">Click to upload photo</p>
                                    </div>
                                `}
                            </div>
                            
                            <input 
                                type="file" 
                                id="logoUpload"
                                accept="image/*" 
                                onchange="app.handleLogoUpload(event)"
                                style="display: none;"
                            />
                            
                            ${s.customLogo ? `
                                <button 
                                    onclick="app.state.customLogo = null; localStorage.removeItem('customLogo'); app.render();"
                                    class="w-full mt-4 px-4 py-3 bg-red-50 hover:bg-red-100 text-red-600 rounded-lg font-semibold transition-colors">
                                    🗑️ Remove Photo
                                </button>
                            ` : ''}
                        </div>
                    </div>
                `;
            },
            
            renderPrayer(next) {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-green-500 to-blue-500 text-white p-6 rounded-xl shadow-lg transform transition-all hover:shadow-2xl">
                            <div class="text-center">
                                <h3 class="text-lg font-semibold opacity-90 mb-2">الصلاة القادمة</h3>
                                <h2 class="text-4xl font-bold mb-2 arabic-text">${this.translatePrayerName(next.name)}</h2>
                                <p class="text-2xl opacity-90 mb-3">${next.time}</p>
                                
                                <!-- Time Remaining -->
                                <div class="bg-white bg-opacity-20 rounded-lg p-4 mt-4">
                                    <p class="text-sm opacity-90 mb-1">الوقت المتبقي</p>
                                    <p class="text-3xl font-bold arabic-text">${s.timeRemaining}</p>
                                </div>
                            </div>
                        </div>

                        <div class="bg-white p-4 rounded-lg shadow-sm border border-gray-200">
                            <div class="flex items-center justify-between mb-3">
                                <div class="flex items-center space-x-3 space-x-reverse">
                                    ${this.icon('bell')}
                                    <span class="font-medium arabic-text">تنبيهات الصلاة</span>
                                </div>
                                <button onclick="app.toggleNotifications()" 
                                    class="relative inline-flex h-6 w-11 items-center rounded-full transition-colors ${s.notificationsEnabled ? 'bg-green-600' : 'bg-gray-200'}">
                                    <span class="inline-block h-4 w-4 transform rounded-full bg-white transition-transform ${s.notificationsEnabled ? 'translate-x-6' : 'translate-x-1'}"></span>
                                </button>
                            </div>
                            <div class="flex items-center justify-between text-sm pt-3 border-t border-gray-100">
                                <div class="flex items-center space-x-2 space-x-reverse">
                                    ${s.connectionStatus === 'connected' ? `${this.icon('wifi')}<span class="text-green-600 arabic-text">متصل بصيدا</span>` : 
                                      s.connectionStatus === 'connecting' ? `<div class="animate-spin">${this.icon('refresh')}</div><span class="text-blue-600 arabic-text">جاري الاتصال...</span>` :
                                      `${this.icon('wifioff')}<span class="text-red-600 arabic-text">غير متصل</span>`}
                                </div>
                                ${s.lastUpdated ? `<span class="text-gray-500 arabic-text">آخر تحديث: ${s.lastUpdated.toLocaleTimeString('ar-LB', {hour: 'numeric', minute: '2-digit', hour12: true})}</span>` : ''}
                            </div>
                            <button onclick="app.fetchPrayerTimes()" ${s.isLoading ? 'disabled' : ''}
                                class="w-full mt-3 px-4 py-2 bg-blue-50 hover:bg-blue-100 text-blue-600 rounded-lg font-medium transition-colors disabled:opacity-50 arabic-text">
                                ${s.isLoading ? '🔄 جاري التحديث...' : '🔄 تحديث أوقات الصلاة'}
                            </button>
                        </div>

                        <div class="bg-white rounded-lg shadow-sm border border-gray-200 overflow-hidden">
                            <h3 class="text-lg font-semibold p-4 border-b border-gray-200 bg-gray-50 arabic-text text-right">أوقات الصلاة اليوم</h3>
                            <div class="divide-y divide-gray-100">
                                ${Object.entries(s.prayerTimes).map(([p,t]) => `
                                    <div class="flex items-center justify-between p-4 hover:bg-gray-50 transition-colors" dir="rtl">
                                        <div class="flex items-center space-x-3 space-x-reverse">
                                            <div class="w-3 h-3 rounded-full ${p === next.name ? 'bg-green-500 animate-pulse' : 'bg-gray-300'}"></div>
                                            <span class="font-medium text-gray-700 arabic-text text-lg">${this.translatePrayerName(p)}</span>
                                        </div>
                                        <span class="text-lg font-semibold text-gray-900">${t}</span>
                                    </div>
                                `).join('')}
                            </div>
                        </div>
                    </div>
                `;
            },
            
            renderAdhan() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-amber-500 to-orange-500 text-white p-6 rounded-xl shadow-lg">
                            <div class="flex items-center space-x-3 mb-4">
                                ${this.icon('sun')}
                                <h2 class="text-2xl font-bold">Adhan Player</h2>
                            </div>
                            <p class="text-lg opacity-90">Listen to the beautiful call to prayer</p>
                        </div>

                        <!-- Instructions -->
                        <div class="bg-blue-50 p-4 rounded-lg border-2 border-blue-200">
                            <h3 class="font-bold text-blue-800 mb-2">📋 Instructions:</h3>
                            <ol class="text-sm text-blue-700 space-y-1 list-decimal list-inside">
                                <li>Save this HTML file in your <strong>Downloads folder</strong></li>
                                <li>Your MP3 files should be in the <strong>same folder</strong></li>
                                <li>Or click "Choose File" below to upload any MP3</li>
                            </ol>
                        </div>

                        <!-- Upload Audio -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-lg font-semibold mb-4">📁 Upload Your Adhan MP3</h3>
                            <input
                                type="file"
                                accept="audio/*"
                                onchange="app.handleAudioUpload(event)"
                                class="block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-green-50 file:text-green-700 hover:file:bg-green-100 cursor-pointer"
                            />
                            ${s.uploadedAudio ? '<p class="text-green-600 text-sm mt-2">✅ Audio uploaded! Ready to play.</p>' : ''}
                        </div>

                        <!-- Choose Adhan -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-lg font-semibold mb-4">Choose Adhan Source</h3>
                            <div class="space-y-3">
                                ${Object.entries(this.muadhuns).map(([k,m]) => `
                                    <div onclick="app.state.selectedMuadhun='${k}'; app.render()" 
                                        class="p-4 rounded-lg border-2 cursor-pointer transition-all transform hover:scale-102 ${
                                            s.selectedMuadhun === k ? 'border-amber-500 bg-amber-50 shadow-md' : 'border-gray-200 hover:border-gray-300'
                                        }">
                                        <div class="flex items-center justify-between">
                                            <div>
                                                <h4 class="font-semibold text-gray-800">${m.name}</h4>
                                                <p class="text-sm text-gray-600">${m.desc}</p>
                                            </div>
                                            <div class="w-4 h-4 rounded-full border-2 ${s.selectedMuadhun === k ? 'border-amber-500 bg-amber-500' : 'border-gray-300'}"></div>
                                        </div>
                                    </div>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Play Button -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <div class="text-center space-y-4">
                                <h3 class="text-lg font-semibold">${this.muadhuns[s.selectedMuadhun].name}</h3>
                                <button onclick="app.playAdhan()" 
                                    class="px-8 py-4 rounded-full text-white font-semibold text-lg transition-all transform hover:scale-105 shadow-lg ${
                                        s.isPlayingAdhan ? 'bg-red-500 hover:bg-red-600 animate-pulse' : 'bg-gradient-to-r from-green-500 to-blue-500 hover:from-green-600 hover:to-blue-600'
                                    }">
                                    ${s.isPlayingAdhan ? '⏸️ Stop Adhan' : '▶️ Play Adhan'}
                                </button>
                                ${s.isPlayingAdhan ? `
                                    <div class="bg-green-50 p-4 rounded-lg border border-green-200 animate-pulse">
                                        <p class="text-green-700 font-medium">🎵 Playing Adhan...</p>
                                        <p class="text-sm text-gray-600 mt-1">${s.uploadedAudio ? 'Your uploaded audio' : this.muadhuns[s.selectedMuadhun].name}</p>
                                    </div>
                                ` : ''}
                                <p class="text-sm text-gray-500 mt-2">🔊 Make sure your volume is on</p>
                            </div>
                        </div>
                    </div>
                `;
            },
            
            renderAthkar() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-white p-4 rounded-lg shadow-sm border border-gray-200">
                            <div class="flex space-x-2 overflow-x-auto">
                                ${Object.keys(this.athkar).map(cat => `
                                    <button onclick="app.state.selectedAthkar='${cat}'; app.render()" 
                                        class="px-4 py-2 rounded-lg font-medium capitalize transition-all whitespace-nowrap ${
                                            s.selectedAthkar === cat ? 'bg-green-500 text-white shadow-md' : 'bg-gray-100 text-gray-700 hover:bg-gray-200'
                                        }">
                                        ${cat}
                                    </button>
                                `).join('')}
                            </div>
                        </div>

                        ${this.athkar[s.selectedAthkar].map(d => `
                            <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200 transform transition-all hover:shadow-lg">
                                <div class="text-right mb-4">
                                    <p class="text-2xl text-green-700 leading-relaxed">${d.ar}</p>
                                </div>
                                <div class="space-y-2">
                                    <p class="text-gray-600 italic text-sm">${d.tr}</p>
                                    <p class="text-gray-800 font-medium">${d.en}</p>
                                </div>
                            </div>
                        `).join('')}
                    </div>
                `;
            },
            
            renderTawba() {
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-purple-500 to-pink-500 text-white p-6 rounded-xl shadow-lg">
                            <div class="flex items-center space-x-3 space-x-reverse mb-4" dir="rtl">
                                ${this.icon('heart')}
                                <h2 class="text-2xl font-bold arabic-text">التوبة - الاستغفار</h2>
                            </div>
                            <p class="text-lg opacity-90 arabic-text text-right leading-relaxed">
                                "وَمَن تَابَ بَعْدَ ظُلْمِهِ وَأَصْلَحَ فَإِنَّ اللَّهَ يَتُوبُ عَلَيْهِ ۗ إِنَّ اللَّهَ غَفُورٌ رَّحِيمٌ" - سورة المائدة 39
                            </p>
                        </div>

                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-xl font-semibold mb-4 text-center arabic-text">الاستغفار اليومي</h3>
                            <div class="text-center">
                                <div class="text-4xl font-bold text-green-600 mb-4 arabic-text leading-relaxed">
                                    أَسْتَغْفِرُ اللَّهَ الْعَظِيمَ
                                </div>
                                <p class="text-gray-600 mb-2 italic">Astaghfiru Allah al-Adheem</p>
                                <p class="text-sm text-gray-500 mb-4 arabic-text">أستغفر الله العظيم وأتوب إليه</p>
                            </div>
                        </div>

                        <div class="space-y-4">
                            <h3 class="text-xl font-semibold text-gray-700 arabic-text text-right">أدعية التوبة والاستغفار</h3>
                            
                            <div class="bg-gradient-to-r from-blue-50 to-purple-50 p-6 rounded-lg border-l-4 border-purple-400">
                                <p class="text-2xl text-gray-800 arabic-text text-right leading-loose mb-3">
                                    اللَّهُمَّ إِنَّكَ عَفُوٌّ كَرِيمٌ تُحِبُّ الْعَفْوَ فَاعْفُ عَنَّا
                                </p>
                                <p class="text-sm text-gray-600 italic">Allahumma innaka 'afuwwun karimun tuhibbul-'afwa fa'fu 'anna</p>
                                <p class="text-sm text-gray-700 mt-2 arabic-text text-right">
                                    اللهم إنك عفو كريم تحب العفو فاعف عنا
                                </p>
                            </div>

                            <div class="bg-gradient-to-r from-green-50 to-blue-50 p-6 rounded-lg border-l-4 border-green-400">
                                <p class="text-2xl text-gray-800 arabic-text text-right leading-loose mb-3">
                                    رَبَّنَا آتِنَا فِي الدُّنْيَا حَسَنَةً وَفِي الْآخِرَةِ حَسَنَةً وَقِنَا عَذَابَ النَّارِ
                                </p>
                                <p class="text-sm text-gray-600 italic">Rabbana atina fid-dunya hasanatan wa fil-akhirati hasanatan wa qina 'adhaban-nar</p>
                                <p class="text-sm text-gray-700 mt-2 arabic-text text-right">
                                    اللهم آتنا في الدنيا حسنة وفي الآخرة حسنة وقنا عذاب النار
                                </p>
                            </div>

                            <div class="bg-gradient-to-r from-purple-50 to-pink-50 p-6 rounded-lg border-l-4 border-pink-400">
                                <p class="text-2xl text-gray-800 arabic-text text-right leading-loose mb-3">
                                    اللَّهُمَّ آتِ نَفْسِي تَقْوَاهَا وَزَكِّهَا أَنتَ خَيْرُ مَن زَكَّاهَا أَنتَ وَلِيُّهَا وَمَوْلَاهَا
                                </p>
                                <p class="text-sm text-gray-600 italic">Allahumma ati nafsi taqwaha wa zakkiha anta khayru man zakkaha anta waliyyuha wa mawlaha</p>
                                <p class="text-sm text-gray-700 mt-2 arabic-text text-right">
                                    اللهم آت نفسي تقواها وزكها أنت خير من زكاها أنت وليها ومولاها
                                </p>
                            </div>
                        </div>

                        <div class="space-y-4">
                            <h3 class="text-xl font-semibold text-gray-700 arabic-text text-right">تذكيرات يومية</h3>
                            ${this.tawbaReminders.map(r => `
                                <div class="bg-gradient-to-r from-blue-50 to-purple-50 p-4 rounded-lg border-r-4 border-purple-400 transform transition-all hover:shadow-md" dir="rtl">
                                    <p class="text-gray-700 arabic-text text-right">${r}</p>
                                </div>
                            `).join('')}
                        </div>

                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-xl font-semibold mb-4 arabic-text text-right">دعاء الاستغفار</h3>
                            <div class="border-r-4 border-green-400 pr-4" dir="rtl">
                                <p class="text-2xl text-green-700 mb-3 text-right leading-loose arabic-text">
                                    رَبَّنَا ظَلَمْنَا أَنفُسَنَا وَإِن لَّمْ تَغْفِرْ لَنَا وَتَرْحَمْنَا لَنَكُونَنَّ مِنَ الْخَاسِرِينَ
                                </p>
                                <p class="text-sm text-gray-600 italic mb-1">
                                    Rabbana zalamna anfusana wa in lam taghfir lana wa tarhamna lanakoonanna minal-khasireen
                                </p>
                                <p class="text-sm text-gray-700 arabic-text text-right">
                                    "رَبَّنَا ظَلَمْنَا أَنفُسَنَا وَإِن لَّمْ تَغْفِرْ لَنَا وَتَرْحَمْنَا لَنَكُونَنَّ مِنَ الْخَاسِرِينَ"
                                </p>
                            </div>
                        </div>
                    </div>
                `;
            },
            
            init() {
                this.fetchPrayerTimes();
                setInterval(() => {
                    this.state.currentTime = new Date();
                    this.render();
                }, 1000);
                setInterval(() => this.fetchPrayerTimes(), 21600000);
                setInterval(() => {
                    const now = new Date();
                    if(now.getHours() === 0 && now.getMinutes() === 0) {
                        this.fetchPrayerTimes();
                    }
                }, 60000);
                this.render();
            }
        };
        
        window.app = app;
        app.init();
    </script>
</body>
</html><!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <meta name="author" content="Abu Dawod">
    <meta name="description" content="Islamic Prayer Times App for Saida, Lebanon - Made by Abu Dawod">
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="apple-mobile-web-app-title" content="أبو داود">
    <meta name="application-name" content="أبو داود">
    <meta name="theme-color" content="#16a34a">
    
    <!-- PWA Manifest -->
    <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoi2KPYqNmIINiv2KfZiNivIiwic2hvcnRfbmFtZSI6Itin2KjZiCDYr9in2YjYryIsInN0YXJ0X3VybCI6Ii4vaW5kZXguaHRtbCIsImRpc3BsYXkiOiJzdGFuZGFsb25lIiwiYmFja2dyb3VuZF9jb2xvciI6IiNmZmZmZmYiLCJ0aGVtZV9jb2xvciI6IiMxNmEzNGEiLCJpY29ucyI6W3sic3JjIjoiZGF0YTppbWFnZS9zdmcreG1sO2Jhc2U2NCxQSE4yWnlCNGJXeHVjejBpYUhSMGNEb3ZMM2QzZHk1M015NXZjbWN2TWpBd01DOXpkbWNpSUhacFpYZENiM2c5SWpBZ01DQXlOQ0F5TkNJK1BHTnBjbU5zWlNCamVEMGlNVElpSUdONVBTSXhNaUlnY2owSU5pSWdabWxzYkQwaUl6RTJZVE0wWVNJdlBqd3ZjM1puUGc9PSIsInNpemVzIjoiNTEyeDUxMiIsInR5cGUiOiJpbWFnZS9zdmcreG1sIn1dfQ==">
    
    <!-- Favicon / App Icon -->
    <link rel="icon" type="image/svg+xml" href="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI+PGNpcmNsZSBjeD0iMTIiIGN5PSIxMiIgcj0iNiIgZmlsbD0iIzE2YTM0YSIvPjwvc3ZnPg==">
    
    <title>أبو داود - Prayer Times</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .animate-pulse { animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite; }
        @keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: .5; } }
        .animate-spin { animation: spin 1s linear infinite; }
        @keyframes spin { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }
        .transition-all { transition: all 0.3s ease; }
        .transform:hover { transform: scale(1.02); }
        .tab-button { transition: all 0.3s ease; }
        .tab-button:hover { background-color: rgba(0,0,0,0.05); }
    </style>
</head>
<body class="bg-gradient-to-br from-green-50 to-blue-50 min-h-screen">
    <div id="app"></div>

    <script>
        const app = {
            state: {
                currentTime: new Date(),
                activeTab: 'prayer',
                notificationsEnabled: localStorage.getItem('notificationsEnabled') === 'true',
                selectedAthkar: 'morning',
                selectedMuadhun: 'local1',
                isPlayingAdhan: false,
                currentAudio: null,
                uploadedAudio: null,
                prayerTimes: {
                    fajr: '05:30',
                    sunrise: '06:45',
                    dhuhr: '12:15',
                    asr: '15:45',
                    maghrib: '18:20',
                    isha: '19:45'
                },
                isLoading: false,
                lastUpdated: null,
                connectionStatus: 'loading',
                currentHijriDate: '',
                customLogo: localStorage.getItem('customLogo') || null
            },
            
            SAIDA: { latitude: 33.5631, longitude: 35.3708, city: 'Saida', country: 'Lebanon' },
            
            muadhuns: {
                local1: { name: 'Your Adhan 1 (haii3alslah936584)', desc: 'Local MP3 file from Downloads', url: './haii3alslah936584 (1).mp3' },
                local2: { name: 'Your Adhan 2 (haii3alslah2336584)', desc: 'Local MP3 file from Downloads', url: './haii3alslah2336584.mp3' },
                upload: { name: 'Upload Your Own', desc: 'Click to upload any MP3 file', url: null }
            },
            
            athkar: {
                morning: [
                    { ar: 'اللَّهُمَّ أَعِنِّي عَلَى ذِكْرِكَ وَشُكْرِكَ وَحُسْنِ عِبَادَتِكَ', tr: 'Allahumma a\'inni ala dhikrika wa shukrika wa husni ibadatika', en: 'O Allah, help me to remember You, to thank You, and to worship You in the best manner.' },
                    { ar: 'أَصْبَحْنَا وَأَصْبَحَ الْمُلْكُ لِلَّهِ', tr: 'Asbahna wa asbaha al-mulku lillahi', en: 'We have entered the morning and the whole kingdom belongs to Allah.' },
                    { ar: 'اللَّهُمَّ بِكَ أَصْبَحْنَا وَبِكَ أَمْسَيْنَا وَبِكَ نَحْيَا وَبِكَ نَمُوتُ وَإِلَيْكَ النُّشُورُ', tr: 'Allahumma bika asbahna wa bika amsayna wa bika nahya wa bika namutu wa ilayka an-nushur', en: 'O Allah, by You we enter the morning and by You we enter the evening, by You we live and by You we die, and to You is the resurrection.' }
                ],
                evening: [
                    { ar: 'أَمْسَيْنَا وَأَمْسَى الْمُلْكُ لِلَّهِ', tr: 'Amsayna wa amsa al-mulku lillahi', en: 'We have entered the evening and the whole kingdom belongs to Allah.' },
                    { ar: 'اللَّهُمَّ إِنِّي أَعُوذُ بِكَ مِنْ عَذَابِ جَهَنَّمَ وَمِنْ عَذَابِ الْقَبْرِ', tr: 'Allahumma inni a\'udhu bika min adhabi jahannama wa min adhabi al-qabr', en: 'O Allah, I seek refuge in You from the punishment of Hell and from the punishment of the grave.' }
                ],
                general: [
                    { ar: 'سُبْحَانَ اللَّهِ وَبِحَمْدِهِ', tr: 'Subhan Allahi wa bihamdihi', en: 'Glory be to Allah and praise be to Him.' },
                    { ar: 'لَا إِلَهَ إِلَّا اللَّهُ وَحْدَهُ لَا شَرِيكَ لَهُ', tr: 'La ilaha illa Allah wahdahu la sharika lahu', en: 'There is no god but Allah alone, with no partner.' },
                    { ar: 'أَسْتَغْفِرُ اللَّهَ الْعَظِيمَ', tr: 'Astaghfiru Allah al-adheem', en: 'I seek forgiveness from Allah, the Most Great.' }
                ]
            },
            
            tawbaReminders: [
                'Remember to seek Allah\'s forgiveness regularly - He is Most Forgiving, Most Merciful.',
                'Turn to Allah in repentance before it\'s too late. Every moment is a chance for a new beginning.',
                'Allah loves those who turn to Him in repentance and purify themselves.',
                'No sin is too great for Allah\'s forgiveness when you sincerely repent.',
                'Make Istighfar (seeking forgiveness) a daily habit - it brings peace to the heart.'
            ],
            
            async fetchPrayerTimes() {
                this.state.isLoading = true;
                this.state.connectionStatus = 'connecting';
                this.render();
                
                try {
                    const d = new Date();
                    const url = `https://api.aladhan.com/v1/timings/${d.getDate()}-${d.getMonth()+1}-${d.getFullYear()}?latitude=${this.SAIDA.latitude}&longitude=${this.SAIDA.longitude}&method=8`;
                    const res = await fetch(url);
                    const data = await res.json();
                    
                    // Convert 24hr to 12hr format
                    const convert24to12 = (time24) => {
                        const [hours, minutes] = time24.split(':');
                        const h = parseInt(hours);
                        const period = h >= 12 ? 'PM' : 'AM';
                        const h12 = h === 0 ? 12 : h > 12 ? h - 12 : h;
                        return `${h12}:${minutes} ${period}`;
                    };
                    
                    this.state.prayerTimes = {
                        fajr: convert24to12(data.data.timings.Fajr),
                        sunrise: convert24to12(data.data.timings.Sunrise),
                        dhuhr: convert24to12(data.data.timings.Dhuhr),
                        asr: convert24to12(data.data.timings.Asr),
                        maghrib: convert24to12(data.data.timings.Maghrib),
                        isha: convert24to12(data.data.timings.Isha)
                    };
                    
                    this.state.currentHijriDate = `${data.data.date.hijri.day} ${data.data.date.hijri.month.en} ${data.data.date.hijri.year}`;
                    this.state.connectionStatus = 'connected';
                    this.state.lastUpdated = new Date();
                } catch(e) {
                    console.error(e);
                    this.state.connectionStatus = 'offline';
                }
                
                this.state.isLoading = false;
                this.render();
            },
            
            getNextPrayer() {
                const now = new Date();
                const currentHour = now.getHours();
                const currentMin = now.getMinutes();
                const currentTimeInMin = currentHour * 60 + currentMin;
                
                // Convert 12hr format back to minutes for comparison
                const timeToMinutes = (time12) => {
                    const match = time12.match(/(\d+):(\d+)\s*(AM|PM)/i);
                    if (!match) return 0;
                    let hours = parseInt(match[1]);
                    const minutes = parseInt(match[2]);
                    const period = match[3].toUpperCase();
                    
                    if (period === 'PM' && hours !== 12) hours += 12;
                    if (period === 'AM' && hours === 12) hours = 0;
                    
                    return hours * 60 + minutes;
                };
                
                const prayers = [
                    ['fajr', this.state.prayerTimes.fajr],
                    ['dhuhr', this.state.prayerTimes.dhuhr],
                    ['asr', this.state.prayerTimes.asr],
                    ['maghrib', this.state.prayerTimes.maghrib],
                    ['isha', this.state.prayerTimes.isha]
                ];
                
                for(let [prayer, time] of prayers) {
                    const prayerTimeInMin = timeToMinutes(time);
                    if(prayerTimeInMin > currentTimeInMin) {
                        // Calculate time remaining
                        const diffMin = prayerTimeInMin - currentTimeInMin;
                        const hours = Math.floor(diffMin / 60);
                        const minutes = diffMin % 60;
                        
                        let remaining = '';
                        if (hours > 0) {
                            remaining = `${hours} ساعة و ${minutes} دقيقة`;
                        } else {
                            remaining = `${minutes} دقيقة`;
                        }
                        
                        this.state.timeRemaining = remaining;
                        return {name: prayer, time: time};
                    }
                }
                
                // Next day's Fajr
                const fajrTimeInMin = timeToMinutes(this.state.prayerTimes.fajr);
                const diffMin = (24 * 60) - currentTimeInMin + fajrTimeInMin;
                const hours = Math.floor(diffMin / 60);
                const minutes = diffMin % 60;
                this.state.timeRemaining = `${hours} ساعة و ${minutes} دقيقة`;
                
                return {name: 'fajr', time: this.state.prayerTimes.fajr};
            },
            
            
            
            handleLogoUpload(event) {
                const file = event.target.files[0];
                if (file && file.type.startsWith('image/')) {
                    const reader = new FileReader();
                    reader.onload = (e) => {
                        this.state.customLogo = e.target.result;
                        localStorage.setItem('customLogo', e.target.result);
                        alert('✅ Logo uploaded successfully!');
                        this.render();
                    };
                    reader.readAsDataURL(file);
                } else {
                    alert('Please upload a valid image file (PNG, JPG, etc.)');
                }
            },
            
            shareApp() {
                const shareData = {
                    title: 'أبو داود - Islamic Prayer App',
                    text: 'Check out this beautiful Islamic prayer times app for Saida, Lebanon! Features: Daily prayer times, Adhan audio, Athkar, and more. Made by Abu Dawod.',
                    url: window.location.href
                };
                
                if (navigator.share) {
                    navigator.share(shareData)
                        .then(() => {
                            document.getElementById('shareStatus').textContent = '✅ Shared successfully!';
                            setTimeout(() => {
                                document.getElementById('shareStatus').textContent = '';
                            }, 3000);
                        })
                        .catch((error) => {
                            if (error.name !== 'AbortError') {
                                this.copyLink();
                            }
                        });
                } else {
                    this.copyLink();
                }
            },
            
            copyLink() {
                const link = window.location.href;
                
                if (navigator.clipboard && navigator.clipboard.writeText) {
                    navigator.clipboard.writeText(link)
                        .then(() => {
                            document.getElementById('shareStatus').textContent = '✅ Link copied to clipboard!';
                            setTimeout(() => {
                                document.getElementById('shareStatus').textContent = '';
                            }, 3000);
                        })
                        .catch(() => {
                            this.fallbackCopyLink(link);
                        });
                } else {
                    this.fallbackCopyLink(link);
                }
            },
            
            fallbackCopyLink(link) {
                const textArea = document.createElement('textarea');
                textArea.value = link;
                textArea.style.position = 'fixed';
                textArea.style.left = '-999999px';
                document.body.appendChild(textArea);
                textArea.focus();
                textArea.select();
                
                try {
                    document.execCommand('copy');
                    document.getElementById('shareStatus').textContent = '✅ Link copied! Share it with WhatsApp, Telegram, etc.';
                    setTimeout(() => {
                        document.getElementById('shareStatus').textContent = '';
                    }, 5000);
                } catch (err) {
                    document.getElementById('shareStatus').textContent = '⚠️ Please manually copy: ' + link;
                }
                
                document.body.removeChild(textArea);
            },
            
            handleAudioUpload(event) {
                const file = event.target.files[0];
                if (file && file.type.startsWith('audio/')) {
                    const audioUrl = URL.createObjectURL(file);
                    this.state.uploadedAudio = audioUrl;
                    alert('✅ Audio file uploaded successfully! Click Play to listen.');
                    this.render();
                } else {
                    alert('Please upload a valid audio file (MP3, WAV, etc.)');
                }
            },
            
            playAdhan() {
                if(this.state.isPlayingAdhan && this.state.currentAudio) {
                    this.state.currentAudio.pause();
                    this.state.currentAudio.currentTime = 0;
                    this.state.isPlayingAdhan = false;
                    this.state.currentAudio = null;
                    this.render();
                    return;
                }
                
                // Use uploaded audio, or selected muadhun URL
                let audioSource = this.state.uploadedAudio || this.muadhuns[this.state.selectedMuadhun].url;
                
                if (!audioSource) {
                    alert('Please upload an MP3 file first, or make sure the MP3 files are in the same folder as this HTML file.');
                    return;
                }
                
                const audio = new Audio(audioSource);
                
                audio.addEventListener('error', (e) => {
                    console.error('Audio error:', e);
                    alert('⚠️ Cannot load audio file.\n\n✅ INSTRUCTIONS:\n1. Save this HTML file in your Downloads folder\n2. Make sure your MP3 files are in the same folder\n3. Or use the "Upload Audio" button to select any MP3 file');
                    this.state.isPlayingAdhan = false;
                    this.render();
                });
                
                audio.play().then(() => {
                    console.log('✅ Audio playing!');
                    this.state.isPlayingAdhan = true;
                    this.state.currentAudio = audio;
                    this.render();
                }).catch((err) => {
                    console.error('Play error:', err);
                    if (err.name === 'NotAllowedError') {
                        alert('Browser blocked audio! Click anywhere on the page first, then try again.');
                    } else {
                        alert('⚠️ Cannot play audio.\n\n✅ SOLUTION: Use the "Upload Audio" button to select your MP3 file manually.');
                    }
                    this.state.isPlayingAdhan = false;
                    this.render();
                });
                
                audio.onended = () => {
                    this.state.isPlayingAdhan = false;
                    this.state.currentAudio = null;
                    this.render();
                };
            },
            
            async toggleNotifications() {
                if(!this.state.notificationsEnabled) {
                    if('Notification' in window) {
                        const perm = await Notification.requestPermission();
                        if(perm === 'granted') {
                            this.state.notificationsEnabled = true;
                            localStorage.setItem('notificationsEnabled', 'true');
                        }
                    }
                } else {
                    this.state.notificationsEnabled = false;
                    localStorage.setItem('notificationsEnabled', 'false');
                }
                this.render();
            },
            
            icon(name) {
                const icons = {
                    moon: '<path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path>',
                    bell: '<path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"></path><path d="M13.73 21a2 2 0 0 1-3.46 0"></path>',
                    clock: '<circle cx="12" cy="12" r="10"></circle><polyline points="12 6 12 12 16 14"></polyline>',
                    sun: '<circle cx="12" cy="12" r="5"></circle><line x1="12" y1="1" x2="12" y2="3"></line><line x1="12" y1="21" x2="12" y2="23"></line><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line><line x1="1" y1="12" x2="3" y2="12"></line><line x1="21" y1="12" x2="23" y2="12"></line><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line>',
                    book: '<path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20"></path><path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z"></path>',
                    heart: '<path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"></path>',
                    wifi: '<path d="M5 12.55a11 11 0 0 1 14.08 0"></path><path d="M1.42 9a16 16 0 0 1 21.16 0"></path><path d="M8.53 16.11a6 6 0 0 1 6.95 0"></path><line x1="12" y1="20" x2="12.01" y2="20"></line>',
                    wifioff: '<line x1="1" y1="1" x2="23" y2="23"></line><path d="M16.72 11.06A10.94 10.94 0 0 1 19 12.55"></path><path d="M5 12.55a10.94 10.94 0 0 1 5.17-2.39"></path><path d="M10.71 5.05A16 16 0 0 1 22.58 9"></path><path d="M1.42 9a15.91 15.91 0 0 1 4.7-2.88"></path><path d="M8.53 16.11a6 6 0 0 1 6.95 0"></path><line x1="12" y1="20" x2="12.01" y2="20"></line>',
                    refresh: '<polyline points="23 4 23 10 17 10"></polyline><polyline points="1 20 1 14 7 14"></polyline><path d="M3.51 9a9 9 0 0 1 14.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0 0 20.49 15"></path>',
                    pin: '<path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z"></path><circle cx="12" cy="10" r="3"></circle>'
                };
                return `<svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">${icons[name]}</svg>`;
            },
            
            
            translatePrayerName(prayer) {
                const names = {
                    fajr: 'الفجر',
                    sunrise: 'الشروق',
                    dhuhr: 'الظهر',
                    asr: 'العصر',
                    maghrib: 'المغرب',
                    isha: 'العشاء'
                };
                return names[prayer] || prayer;
            },
            
            render() {
                const s = this.state;
                const next = this.getNextPrayer();
                
                document.getElementById('app').innerHTML = `
                    <div class="min-h-screen">
                        <!-- App Header with Logo -->
                        <div class="bg-gradient-to-r from-green-600 to-blue-600 text-white p-6 shadow-lg">
                            <!-- App Logo -->
                            <div class="app-logo arabic-text">أد</div>
                            
                            <div class="flex items-center justify-between flex-wrap gap-4">
                                <div class="flex items-center space-x-3">
                                    <div>
                                        <h1 class="text-2xl font-bold arabic-text">أبو داود</h1>
                                        <p class="text-sm text-green-100">Prayer Times - Saida, Lebanon</p>
                                    </div>
                                </div>
                                <div class="text-right">
                                    <div class="text-2xl font-bold">${s.currentTime.toLocaleTimeString('en-US', {hour: 'numeric', minute: '2-digit', hour12: true})}</div>
                                    <div class="text-sm opacity-90">${s.currentTime.toLocaleDateString()}</div>
                                    ${s.currentHijriDate ? `<div class="text-xs opacity-75 mt-1 arabic-text">${s.currentHijriDate}</div>` : ''}
                                </div>
                            </div>
                            
                            <!-- Custom Logo Section -->
                            ${s.customLogo ? `
                                <div class="mt-6 text-center">
                                    <img src="${s.customLogo}" class="custom-logo-img mx-auto" style="max-height: 120px; border-radius: 15px; box-shadow: 0 5px 15px rgba(0,0,0,0.2);" alt="App Logo">
                                </div>
                            ` : ''}
                        </div>

                        <!-- Tabs -->
                        <div class="bg-white shadow-sm">
                            <div class="flex overflow-x-auto" dir="rtl">
                                ${[
                                    {id: 'prayer', label: 'الصلاة'},
                                    {id: 'adhan', label: 'الأذان'},
                                    {id: 'athkar', label: 'الأذكار'},
                                    {id: 'tawba', label: 'التوبة'},
                                    {id: 'profile', label: '👤 الملف'}
                                ].map(tab => `
                                    <button onclick="app.state.activeTab='${tab.id}'; app.render()" 
                                        class="tab-button flex-1 flex items-center justify-center space-x-2 py-4 px-6 border-b-2 transition-colors whitespace-nowrap ${
                                            s.activeTab === tab.id ? 'border-green-500 text-green-600 bg-green-50' : 'border-transparent text-gray-500 hover:text-gray-700'
                                        }">
                                        <span class="font-medium arabic-text">${tab.label}</span>
                                    </button>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Made by Abu Dawod Banner -->
                        <div class="p-6 pb-4">
                            <div class="bg-gradient-to-r from-green-600 to-blue-600 text-white p-4 rounded-xl shadow-lg text-center">
                                <p class="text-lg font-semibold arabic-text">صُنع بـ ❤️ من قبل أبو داود</p>
                                <p class="text-sm opacity-90">May Allah accept this work and make it beneficial for all Muslims</p>
                            </div>
                        </div>

                        <!-- Content -->
                        <div class="p-6 pt-0">
                            ${s.activeTab === 'prayer' ? this.renderPrayer(next) : ''}
                            ${s.activeTab === 'adhan' ? this.renderAdhan() : ''}
                            ${s.activeTab === 'athkar' ? this.renderAthkar() : ''}
                            ${s.activeTab === 'tawba' ? this.renderTawba() : ''}
                            ${s.activeTab === 'profile' ? this.renderProfile() : ''}
                        </div>
                    </div>
                `;
            },
            
            renderProfile() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <!-- Developer Profile Header -->
                        <div class="bg-gradient-to-r from-green-600 via-emerald-600 to-blue-600 text-white p-8 rounded-xl shadow-2xl">
                            <div class="text-center">
                                <!-- Profile Picture Upload Area -->
                                <div class="mb-6">
                                    <div class="w-32 h-32 mx-auto rounded-full bg-white bg-opacity-20 flex items-center justify-center cursor-pointer hover:bg-opacity-30 transition-all overflow-hidden border-4 border-white shadow-lg" onclick="document.getElementById('logoUpload').click()">
                                        ${s.customLogo ? `
                                            <img src="${s.customLogo}" class="w-full h-full object-cover" alt="Profile">
                                        ` : `
                                            <svg class="w-16 h-16 text-white opacity-75" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"></path>
                                            </svg>
                                        `}
                                    </div>
                                    <input 
                                        type="file" 
                                        id="logoUpload"
                                        accept="image/*" 
                                        onchange="app.handleLogoUpload(event)"
                                        style="display: none;"
                                    />
                                    <p class="text-sm mt-2 opacity-75">Click to upload profile photo</p>
                                </div>
                                
                                <h1 class="text-4xl font-bold mb-2 arabic-text">أبو داود</h1>
                                <p class="text-xl mb-2">Abu Dawod</p>
                                <p class="text-lg opacity-90">Islamic App Developer</p>
                                <div class="flex items-center justify-center gap-2 mt-4">
                                    <span class="px-4 py-1 bg-white bg-opacity-20 rounded-full text-sm">Saida, Lebanon 🇱🇧</span>
                                </div>
                            </div>
                        </div>

                        <!-- About This Project -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">📱 About This Application</h2>
                            <div class="space-y-4 text-gray-700">
                                <p class="leading-relaxed">
                                    <strong class="text-green-600">Islamic Prayer Times & Athkar App</strong> is a comprehensive web application designed to help Muslims stay connected with their daily prayers and remembrance of Allah.
                                </p>
                                <p class="leading-relaxed">
                                    This app provides accurate prayer times for Saida, Lebanon, beautiful Adhan audio, daily Athkar, and reminders for Tawba (repentance).
                                </p>
                            </div>
                        </div>

                        <!-- Features -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">✨ Key Features</h2>
                            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                                ${[
                                    { icon: '🕌', title: 'Daily Prayer Times', desc: 'Accurate times for Saida updated automatically' },
                                    { icon: '🔔', title: 'Prayer Notifications', desc: 'Get notified when it\'s time to pray' },
                                    { icon: '🎵', title: 'Beautiful Adhan', desc: 'Multiple Adhan styles with audio playback' },
                                    { icon: '📿', title: 'Daily Athkar', desc: 'Morning, evening, and general remembrances' },
                                    { icon: '💜', title: 'Tawba Reminders', desc: 'Encouragement for repentance and forgiveness' },
                                    { icon: '🌙', title: 'Hijri Calendar', desc: 'Display Islamic date alongside Gregorian' },
                                    { icon: '📱', title: 'Installable App', desc: 'Works like a native mobile application' },
                                    { icon: '🔄', title: 'Auto-Updates', desc: 'Prayer times update daily automatically' }
                                ].map(f => `
                                    <div class="flex items-start space-x-3 p-3 bg-green-50 rounded-lg">
                                        <span class="text-2xl">${f.icon}</span>
                                        <div>
                                            <h3 class="font-semibold text-gray-800">${f.title}</h3>
                                            <p class="text-sm text-gray-600">${f.desc}</p>
                                        </div>
                                    </div>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Technologies Used -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">🛠️ Technologies Used</h2>
                            <div class="flex flex-wrap gap-3">
                                ${['HTML5', 'CSS3', 'JavaScript', 'Tailwind CSS', 'AlAdhan API', 'Web Audio API', 'Local Storage', 'PWA'].map(tech => `
                                    <span class="px-4 py-2 bg-gradient-to-r from-green-100 to-blue-100 text-gray-800 rounded-full text-sm font-semibold border border-green-200">${tech}</span>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Statistics -->
                        <div class="bg-gradient-to-r from-purple-50 to-pink-50 p-6 rounded-xl shadow-lg border-2 border-purple-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">📊 App Statistics</h2>
                            <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
                                ${[
                                    { num: '5', label: 'Daily Prayers' },
                                    { num: '10+', label: 'Athkar Duas' },
                                    { num: '4', label: 'Adhan Styles' },
                                    { num: '100%', label: 'Free & Open' }
                                ].map(stat => `
                                    <div class="text-center p-4 bg-white rounded-lg shadow-sm">
                                        <div class="text-3xl font-bold text-green-600">${stat.num}</div>
                                        <div class="text-sm text-gray-600 mt-1">${stat.label}</div>
                                    </div>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Contact & Share -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">🤝 Share This App</h2>
                            <p class="text-gray-700 mb-4">Help spread the benefit! Share this Islamic prayer app with your community:</p>
                            <div class="space-y-3">
                                <button onclick="app.shareApp()" 
                                    class="w-full px-6 py-3 bg-gradient-to-r from-green-500 to-blue-500 text-white rounded-lg font-semibold hover:from-green-600 hover:to-blue-600 transition-all shadow-md">
                                    📤 Share App
                                </button>
                                <button onclick="app.copyLink()" 
                                    class="w-full px-6 py-3 bg-gray-100 text-gray-800 rounded-lg font-semibold hover:bg-gray-200 transition-all">
                                    📋 Copy App Link
                                </button>
                                <div class="text-center text-sm text-gray-500 mt-2">
                                    <p id="shareStatus"></p>
                                </div>
                            </div>
                        </div>

                        <!-- Credits & License -->
                        <div class="bg-gradient-to-r from-green-600 to-blue-600 text-white p-6 rounded-xl shadow-lg text-center">
                            <h2 class="text-2xl font-bold mb-3 arabic-text">بارك الله فيك</h2>
                            <p class="text-lg mb-4">May Allah bless this work and make it a source of continuous reward (Sadaqah Jariyah)</p>
                            <div class="text-sm opacity-90">
                                <p>Developed with ❤️ by <strong class="arabic-text">أبو داود</strong></p>
                                <p class="mt-2">Version 1.0.0 | 2025</p>
                                <p class="mt-2">Free for all Muslims to use and benefit from</p>
                            </div>
                        </div>

                        ${s.customLogo ? `
                            <div class="bg-white p-4 rounded-lg shadow-sm border border-gray-200 text-center">
                                <button 
                                    onclick="app.state.customLogo = null; localStorage.removeItem('customLogo'); app.render();"
                                    class="px-6 py-2 bg-red-50 hover:bg-red-100 text-red-600 rounded-lg font-medium transition-colors">
                                    🗑️ Remove Profile Photo
                                </button>
                            </div>
                        ` : ''}
                    </div>
                `;
            },
            
            renderSettings() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-purple-500 to-indigo-500 text-white p-6 rounded-xl shadow-lg">
                            <h2 class="text-2xl font-bold mb-2">⚙️ Settings</h2>
                            <p class="text-lg opacity-90">Upload your program image</p>
                        </div>

                        <!-- Upload Photo Only -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-lg font-semibold mb-4 text-center">📸 Program Image</h3>
                            
                            <div class="custom-logo-container" onclick="document.getElementById('logoUpload').click()">
                                ${s.customLogo ? `
                                    <img src="${s.customLogo}" class="custom-logo-img" alt="Program Image">
                                ` : `
                                    <div class="text-center text-gray-400">
                                        <svg class="w-16 h-16 mx-auto mb-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"></path>
                                        </svg>
                                        <p class="text-sm font-semibold">Click to upload photo</p>
                                    </div>
                                `}
                            </div>
                            
                            <input 
                                type="file" 
                                id="logoUpload"
                                accept="image/*" 
                                onchange="app.handleLogoUpload(event)"
                                style="display: none;"
                            />
                            
                            ${s.customLogo ? `
                                <button 
                                    onclick="app.state.customLogo = null; localStorage.removeItem('customLogo'); app.render();"
                                    class="w-full mt-4 px-4 py-3 bg-red-50 hover:bg-red-100 text-red-600 rounded-lg font-semibold transition-colors">
                                    🗑️ Remove Photo
                                </button>
                            ` : ''}
                        </div>
                    </div>
                `;
            },
            
            renderPrayer(next) {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-green-500 to-blue-500 text-white p-6 rounded-xl shadow-lg transform transition-all hover:shadow-2xl">
                            <div class="text-center">
                                <h3 class="text-lg font-semibold opacity-90 mb-2">الصلاة القادمة</h3>
                                <h2 class="text-4xl font-bold mb-2 arabic-text">${this.translatePrayerName(next.name)}</h2>
                                <p class="text-2xl opacity-90 mb-3">${next.time}</p>
                                
                                <!-- Time Remaining -->
                                <div class="bg-white bg-opacity-20 rounded-lg p-4 mt-4">
                                    <p class="text-sm opacity-90 mb-1">الوقت المتبقي</p>
                                    <p class="text-3xl font-bold arabic-text">${s.timeRemaining}</p>
                                </div>
                            </div>
                        </div>

                        <div class="bg-white p-4 rounded-lg shadow-sm border border-gray-200">
                            <div class="flex items-center justify-between mb-3">
                                <div class="flex items-center space-x-3 space-x-reverse">
                                    ${this.icon('bell')}
                                    <span class="font-medium arabic-text">تنبيهات الصلاة</span>
                                </div>
                                <button onclick="app.toggleNotifications()" 
                                    class="relative inline-flex h-6 w-11 items-center rounded-full transition-colors ${s.notificationsEnabled ? 'bg-green-600' : 'bg-gray-200'}">
                                    <span class="inline-block h-4 w-4 transform rounded-full bg-white transition-transform ${s.notificationsEnabled ? 'translate-x-6' : 'translate-x-1'}"></span>
                                </button>
                            </div>
                            <div class="flex items-center justify-between text-sm pt-3 border-t border-gray-100">
                                <div class="flex items-center space-x-2 space-x-reverse">
                                    ${s.connectionStatus === 'connected' ? `${this.icon('wifi')}<span class="text-green-600 arabic-text">متصل بصيدا</span>` : 
                                      s.connectionStatus === 'connecting' ? `<div class="animate-spin">${this.icon('refresh')}</div><span class="text-blue-600 arabic-text">جاري الاتصال...</span>` :
                                      `${this.icon('wifioff')}<span class="text-red-600 arabic-text">غير متصل</span>`}
                                </div>
                                ${s.lastUpdated ? `<span class="text-gray-500 arabic-text">آخر تحديث: ${s.lastUpdated.toLocaleTimeString('ar-LB', {hour: 'numeric', minute: '2-digit', hour12: true})}</span>` : ''}
                            </div>
                            <button onclick="app.fetchPrayerTimes()" ${s.isLoading ? 'disabled' : ''}
                                class="w-full mt-3 px-4 py-2 bg-blue-50 hover:bg-blue-100 text-blue-600 rounded-lg font-medium transition-colors disabled:opacity-50 arabic-text">
                                ${s.isLoading ? '🔄 جاري التحديث...' : '🔄 تحديث أوقات الصلاة'}
                            </button>
                        </div>

                        <div class="bg-white rounded-lg shadow-sm border border-gray-200 overflow-hidden">
                            <h3 class="text-lg font-semibold p-4 border-b border-gray-200 bg-gray-50 arabic-text text-right">أوقات الصلاة اليوم</h3>
                            <div class="divide-y divide-gray-100">
                                ${Object.entries(s.prayerTimes).map(([p,t]) => `
                                    <div class="flex items-center justify-between p-4 hover:bg-gray-50 transition-colors" dir="rtl">
                                        <div class="flex items-center space-x-3 space-x-reverse">
                                            <div class="w-3 h-3 rounded-full ${p === next.name ? 'bg-green-500 animate-pulse' : 'bg-gray-300'}"></div>
                                            <span class="font-medium text-gray-700 arabic-text text-lg">${this.translatePrayerName(p)}</span>
                                        </div>
                                        <span class="text-lg font-semibold text-gray-900">${t}</span>
                                    </div>
                                `).join('')}
                            </div>
                        </div>
                    </div>
                `;
            },
            
            renderAdhan() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-amber-500 to-orange-500 text-white p-6 rounded-xl shadow-lg">
                            <div class="flex items-center space-x-3 mb-4">
                                ${this.icon('sun')}
                                <h2 class="text-2xl font-bold">Adhan Player</h2>
                            </div>
                            <p class="text-lg opacity-90">Listen to the beautiful call to prayer</p>
                        </div>

                        <!-- Instructions -->
                        <div class="bg-blue-50 p-4 rounded-lg border-2 border-blue-200">
                            <h3 class="font-bold text-blue-800 mb-2">📋 Instructions:</h3>
                            <ol class="text-sm text-blue-700 space-y-1 list-decimal list-inside">
                                <li>Save this HTML file in your <strong>Downloads folder</strong></li>
                                <li>Your MP3 files should be in the <strong>same folder</strong></li>
                                <li>Or click "Choose File" below to upload any MP3</li>
                            </ol>
                        </div>

                        <!-- Upload Audio -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-lg font-semibold mb-4">📁 Upload Your Adhan MP3</h3>
                            <input
                                type="file"
                                accept="audio/*"
                                onchange="app.handleAudioUpload(event)"
                                class="block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-green-50 file:text-green-700 hover:file:bg-green-100 cursor-pointer"
                            />
                            ${s.uploadedAudio ? '<p class="text-green-600 text-sm mt-2">✅ Audio uploaded! Ready to play.</p>' : ''}
                        </div>

                        <!-- Choose Adhan -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-lg font-semibold mb-4">Choose Adhan Source</h3>
                            <div class="space-y-3">
                                ${Object.entries(this.muadhuns).map(([k,m]) => `
                                    <div onclick="app.state.selectedMuadhun='${k}'; app.render()" 
                                        class="p-4 rounded-lg border-2 cursor-pointer transition-all transform hover:scale-102 ${
                                            s.selectedMuadhun === k ? 'border-amber-500 bg-amber-50 shadow-md' : 'border-gray-200 hover:border-gray-300'
                                        }">
                                        <div class="flex items-center justify-between">
                                            <div>
                                                <h4 class="font-semibold text-gray-800">${m.name}</h4>
                                                <p class="text-sm text-gray-600">${m.desc}</p>
                                            </div>
                                            <div class="w-4 h-4 rounded-full border-2 ${s.selectedMuadhun === k ? 'border-amber-500 bg-amber-500' : 'border-gray-300'}"></div>
                                        </div>
                                    </div>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Play Button -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <div class="text-center space-y-4">
                                <h3 class="text-lg font-semibold">${this.muadhuns[s.selectedMuadhun].name}</h3>
                                <button onclick="app.playAdhan()" 
                                    class="px-8 py-4 rounded-full text-white font-semibold text-lg transition-all transform hover:scale-105 shadow-lg ${
                                        s.isPlayingAdhan ? 'bg-red-500 hover:bg-red-600 animate-pulse' : 'bg-gradient-to-r from-green-500 to-blue-500 hover:from-green-600 hover:to-blue-600'
                                    }">
                                    ${s.isPlayingAdhan ? '⏸️ Stop Adhan' : '▶️ Play Adhan'}
                                </button>
                                ${s.isPlayingAdhan ? `
                                    <div class="bg-green-50 p-4 rounded-lg border border-green-200 animate-pulse">
                                        <p class="text-green-700 font-medium">🎵 Playing Adhan...</p>
                                        <p class="text-sm text-gray-600 mt-1">${s.uploadedAudio ? 'Your uploaded audio' : this.muadhuns[s.selectedMuadhun].name}</p>
                                    </div>
                                ` : ''}
                                <p class="text-sm text-gray-500 mt-2">🔊 Make sure your volume is on</p>
                            </div>
                        </div>
                    </div>
                `;
            },
            
            renderAthkar() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-white p-4 rounded-lg shadow-sm border border-gray-200">
                            <div class="flex space-x-2 overflow-x-auto">
                                ${Object.keys(this.athkar).map(cat => `
                                    <button onclick="app.state.selectedAthkar='${cat}'; app.render()" 
                                        class="px-4 py-2 rounded-lg font-medium capitalize transition-all whitespace-nowrap ${
                                            s.selectedAthkar === cat ? 'bg-green-500 text-white shadow-md' : 'bg-gray-100 text-gray-700 hover:bg-gray-200'
                                        }">
                                        ${cat}
                                    </button>
                                `).join('')}
                            </div>
                        </div>

                        ${this.athkar[s.selectedAthkar].map(d => `
                            <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200 transform transition-all hover:shadow-lg">
                                <div class="text-right mb-4">
                                    <p class="text-2xl text-green-700 leading-relaxed">${d.ar}</p>
                                </div>
                                <div class="space-y-2">
                                    <p class="text-gray-600 italic text-sm">${d.tr}</p>
                                    <p class="text-gray-800 font-medium">${d.en}</p>
                                </div>
                            </div>
                        `).join('')}
                    </div>
                `;
            },
            
            renderTawba() {
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-purple-500 to-pink-500 text-white p-6 rounded-xl shadow-lg">
                            <div class="flex items-center space-x-3 space-x-reverse mb-4" dir="rtl">
                                ${this.icon('heart')}
                                <h2 class="text-2xl font-bold arabic-text">التوبة - الاستغفار</h2>
                            </div>
                            <p class="text-lg opacity-90 arabic-text text-right leading-relaxed">
                                "وَمَن تَابَ بَعْدَ ظُلْمِهِ وَأَصْلَحَ فَإِنَّ اللَّهَ يَتُوبُ عَلَيْهِ ۗ إِنَّ اللَّهَ غَفُورٌ رَّحِيمٌ" - سورة المائدة 39
                            </p>
                        </div>

                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-xl font-semibold mb-4 text-center arabic-text">الاستغفار اليومي</h3>
                            <div class="text-center">
                                <div class="text-4xl font-bold text-green-600 mb-4 arabic-text leading-relaxed">
                                    أَسْتَغْفِرُ اللَّهَ الْعَظِيمَ
                                </div>
                                <p class="text-gray-600 mb-2 italic">Astaghfiru Allah al-Adheem</p>
                                <p class="text-sm text-gray-500 mb-4 arabic-text">أستغفر الله العظيم وأتوب إليه</p>
                            </div>
                        </div>

                        <div class="space-y-4">
                            <h3 class="text-xl font-semibold text-gray-700 arabic-text text-right">أدعية التوبة والاستغفار</h3>
                            
                            <div class="bg-gradient-to-r from-blue-50 to-purple-50 p-6 rounded-lg border-l-4 border-purple-400">
                                <p class="text-2xl text-gray-800 arabic-text text-right leading-loose mb-3">
                                    اللَّهُمَّ إِنَّكَ عَفُوٌّ كَرِيمٌ تُحِبُّ الْعَفْوَ فَاعْفُ عَنَّا
                                </p>
                                <p class="text-sm text-gray-600 italic">Allahumma innaka 'afuwwun karimun tuhibbul-'afwa fa'fu 'anna</p>
                                <p class="text-sm text-gray-700 mt-2 arabic-text text-right">
                                    اللهم إنك عفو كريم تحب العفو فاعف عنا
                                </p>
                            </div>

                            <div class="bg-gradient-to-r from-green-50 to-blue-50 p-6 rounded-lg border-l-4 border-green-400">
                                <p class="text-2xl text-gray-800 arabic-text text-right leading-loose mb-3">
                                    رَبَّنَا آتِنَا فِي الدُّنْيَا حَسَنَةً وَفِي الْآخِرَةِ حَسَنَةً وَقِنَا عَذَابَ النَّارِ
                                </p>
                                <p class="text-sm text-gray-600 italic">Rabbana atina fid-dunya hasanatan wa fil-akhirati hasanatan wa qina 'adhaban-nar</p>
                                <p class="text-sm text-gray-700 mt-2 arabic-text text-right">
                                    اللهم آتنا في الدنيا حسنة وفي الآخرة حسنة وقنا عذاب النار
                                </p>
                            </div>

                            <div class="bg-gradient-to-r from-purple-50 to-pink-50 p-6 rounded-lg border-l-4 border-pink-400">
                                <p class="text-2xl text-gray-800 arabic-text text-right leading-loose mb-3">
                                    اللَّهُمَّ آتِ نَفْسِي تَقْوَاهَا وَزَكِّهَا أَنتَ خَيْرُ مَن زَكَّاهَا أَنتَ وَلِيُّهَا وَمَوْلَاهَا
                                </p>
                                <p class="text-sm text-gray-600 italic">Allahumma ati nafsi taqwaha wa zakkiha anta khayru man zakkaha anta waliyyuha wa mawlaha</p>
                                <p class="text-sm text-gray-700 mt-2 arabic-text text-right">
                                    اللهم آت نفسي تقواها وزكها أنت خير من زكاها أنت وليها ومولاها
                                </p>
                            </div>
                        </div>

                        <div class="space-y-4">
                            <h3 class="text-xl font-semibold text-gray-700 arabic-text text-right">تذكيرات يومية</h3>
                            ${this.tawbaReminders.map(r => `
                                <div class="bg-gradient-to-r from-blue-50 to-purple-50 p-4 rounded-lg border-r-4 border-purple-400 transform transition-all hover:shadow-md" dir="rtl">
                                    <p class="text-gray-700 arabic-text text-right">${r}</p>
                                </div>
                            `).join('')}
                        </div>

                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-xl font-semibold mb-4 arabic-text text-right">دعاء الاستغفار</h3>
                            <div class="border-r-4 border-green-400 pr-4" dir="rtl">
                                <p class="text-2xl text-green-700 mb-3 text-right leading-loose arabic-text">
                                    رَبَّنَا ظَلَمْنَا أَنفُسَنَا وَإِن لَّمْ تَغْفِرْ لَنَا وَتَرْحَمْنَا لَنَكُونَنَّ مِنَ الْخَاسِرِينَ
                                </p>
                                <p class="text-sm text-gray-600 italic mb-1">
                                    Rabbana zalamna anfusana wa in lam taghfir lana wa tarhamna lanakoonanna minal-khasireen
                                </p>
                                <p class="text-sm text-gray-700 arabic-text text-right">
                                    "رَبَّنَا ظَلَمْنَا أَنفُسَنَا وَإِن لَّمْ تَغْفِرْ لَنَا وَتَرْحَمْنَا لَنَكُونَنَّ مِنَ الْخَاسِرِينَ"
                                </p>
                            </div>
                        </div>
                    </div>
                `;
            },
            
            init() {
                this.fetchPrayerTimes();
                setInterval(() => {
                    this.state.currentTime = new Date();
                    this.render();
                }, 1000);
                setInterval(() => this.fetchPrayerTimes(), 21600000);
                setInterval(() => {
                    const now = new Date();
                    if(now.getHours() === 0 && now.getMinutes() === 0) {
                        this.fetchPrayerTimes();
                    }
                }, 60000);
                this.render();
            }
        };
        
        window.app = app;
        app.init();
    </script>
</body>
</html><!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <meta name="author" content="Abu Dawod">
    <meta name="description" content="Islamic Prayer Times App for Saida, Lebanon - Made by Abu Dawod">
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="apple-mobile-web-app-title" content="أبو داود">
    <meta name="application-name" content="أبو داود">
    <meta name="theme-color" content="#16a34a">
    
    <!-- PWA Manifest -->
    <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoi2KPYqNmIINiv2KfZiNivIiwic2hvcnRfbmFtZSI6Itin2KjZiCDYr9in2YjYryIsInN0YXJ0X3VybCI6Ii4vaW5kZXguaHRtbCIsImRpc3BsYXkiOiJzdGFuZGFsb25lIiwiYmFja2dyb3VuZF9jb2xvciI6IiNmZmZmZmYiLCJ0aGVtZV9jb2xvciI6IiMxNmEzNGEiLCJpY29ucyI6W3sic3JjIjoiZGF0YTppbWFnZS9zdmcreG1sO2Jhc2U2NCxQSE4yWnlCNGJXeHVjejBpYUhSMGNEb3ZMM2QzZHk1M015NXZjbWN2TWpBd01DOXpkbWNpSUhacFpYZENiM2c5SWpBZ01DQXlOQ0F5TkNJK1BHTnBjbU5zWlNCamVEMGlNVElpSUdONVBTSXhNaUlnY2owSU5pSWdabWxzYkQwaUl6RTJZVE0wWVNJdlBqd3ZjM1puUGc9PSIsInNpemVzIjoiNTEyeDUxMiIsInR5cGUiOiJpbWFnZS9zdmcreG1sIn1dfQ==">
    
    <!-- Favicon / App Icon -->
    <link rel="icon" type="image/svg+xml" href="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI+PGNpcmNsZSBjeD0iMTIiIGN5PSIxMiIgcj0iNiIgZmlsbD0iIzE2YTM0YSIvPjwvc3ZnPg==">
    
    <title>أبو داود - Prayer Times</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .animate-pulse { animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite; }
        @keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: .5; } }
        .animate-spin { animation: spin 1s linear infinite; }
        @keyframes spin { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }
        .transition-all { transition: all 0.3s ease; }
        .transform:hover { transform: scale(1.02); }
        .tab-button { transition: all 0.3s ease; }
        .tab-button:hover { background-color: rgba(0,0,0,0.05); }
    </style>
</head>
<body class="bg-gradient-to-br from-green-50 to-blue-50 min-h-screen">
    <div id="app"></div>

    <script>
        const app = {
            state: {
                currentTime: new Date(),
                activeTab: 'prayer',
                notificationsEnabled: localStorage.getItem('notificationsEnabled') === 'true',
                selectedAthkar: 'morning',
                selectedMuadhun: 'local1',
                isPlayingAdhan: false,
                currentAudio: null,
                uploadedAudio: null,
                prayerTimes: {
                    fajr: '05:30',
                    sunrise: '06:45',
                    dhuhr: '12:15',
                    asr: '15:45',
                    maghrib: '18:20',
                    isha: '19:45'
                },
                isLoading: false,
                lastUpdated: null,
                connectionStatus: 'loading',
                currentHijriDate: '',
                customLogo: localStorage.getItem('customLogo') || null
            },
            
            SAIDA: { latitude: 33.5631, longitude: 35.3708, city: 'Saida', country: 'Lebanon' },
            
            muadhuns: {
                local1: { name: 'Your Adhan 1 (haii3alslah936584)', desc: 'Local MP3 file from Downloads', url: './haii3alslah936584 (1).mp3' },
                local2: { name: 'Your Adhan 2 (haii3alslah2336584)', desc: 'Local MP3 file from Downloads', url: './haii3alslah2336584.mp3' },
                upload: { name: 'Upload Your Own', desc: 'Click to upload any MP3 file', url: null }
            },
            
            athkar: {
                morning: [
                    { ar: 'اللَّهُمَّ أَعِنِّي عَلَى ذِكْرِكَ وَشُكْرِكَ وَحُسْنِ عِبَادَتِكَ', tr: 'Allahumma a\'inni ala dhikrika wa shukrika wa husni ibadatika', en: 'O Allah, help me to remember You, to thank You, and to worship You in the best manner.' },
                    { ar: 'أَصْبَحْنَا وَأَصْبَحَ الْمُلْكُ لِلَّهِ', tr: 'Asbahna wa asbaha al-mulku lillahi', en: 'We have entered the morning and the whole kingdom belongs to Allah.' },
                    { ar: 'اللَّهُمَّ بِكَ أَصْبَحْنَا وَبِكَ أَمْسَيْنَا وَبِكَ نَحْيَا وَبِكَ نَمُوتُ وَإِلَيْكَ النُّشُورُ', tr: 'Allahumma bika asbahna wa bika amsayna wa bika nahya wa bika namutu wa ilayka an-nushur', en: 'O Allah, by You we enter the morning and by You we enter the evening, by You we live and by You we die, and to You is the resurrection.' }
                ],
                evening: [
                    { ar: 'أَمْسَيْنَا وَأَمْسَى الْمُلْكُ لِلَّهِ', tr: 'Amsayna wa amsa al-mulku lillahi', en: 'We have entered the evening and the whole kingdom belongs to Allah.' },
                    { ar: 'اللَّهُمَّ إِنِّي أَعُوذُ بِكَ مِنْ عَذَابِ جَهَنَّمَ وَمِنْ عَذَابِ الْقَبْرِ', tr: 'Allahumma inni a\'udhu bika min adhabi jahannama wa min adhabi al-qabr', en: 'O Allah, I seek refuge in You from the punishment of Hell and from the punishment of the grave.' }
                ],
                general: [
                    { ar: 'سُبْحَانَ اللَّهِ وَبِحَمْدِهِ', tr: 'Subhan Allahi wa bihamdihi', en: 'Glory be to Allah and praise be to Him.' },
                    { ar: 'لَا إِلَهَ إِلَّا اللَّهُ وَحْدَهُ لَا شَرِيكَ لَهُ', tr: 'La ilaha illa Allah wahdahu la sharika lahu', en: 'There is no god but Allah alone, with no partner.' },
                    { ar: 'أَسْتَغْفِرُ اللَّهَ الْعَظِيمَ', tr: 'Astaghfiru Allah al-adheem', en: 'I seek forgiveness from Allah, the Most Great.' }
                ]
            },
            
            tawbaReminders: [
                'Remember to seek Allah\'s forgiveness regularly - He is Most Forgiving, Most Merciful.',
                'Turn to Allah in repentance before it\'s too late. Every moment is a chance for a new beginning.',
                'Allah loves those who turn to Him in repentance and purify themselves.',
                'No sin is too great for Allah\'s forgiveness when you sincerely repent.',
                'Make Istighfar (seeking forgiveness) a daily habit - it brings peace to the heart.'
            ],
            
            async fetchPrayerTimes() {
                this.state.isLoading = true;
                this.state.connectionStatus = 'connecting';
                this.render();
                
                try {
                    const d = new Date();
                    const url = `https://api.aladhan.com/v1/timings/${d.getDate()}-${d.getMonth()+1}-${d.getFullYear()}?latitude=${this.SAIDA.latitude}&longitude=${this.SAIDA.longitude}&method=8`;
                    const res = await fetch(url);
                    const data = await res.json();
                    
                    // Convert 24hr to 12hr format
                    const convert24to12 = (time24) => {
                        const [hours, minutes] = time24.split(':');
                        const h = parseInt(hours);
                        const period = h >= 12 ? 'PM' : 'AM';
                        const h12 = h === 0 ? 12 : h > 12 ? h - 12 : h;
                        return `${h12}:${minutes} ${period}`;
                    };
                    
                    this.state.prayerTimes = {
                        fajr: convert24to12(data.data.timings.Fajr),
                        sunrise: convert24to12(data.data.timings.Sunrise),
                        dhuhr: convert24to12(data.data.timings.Dhuhr),
                        asr: convert24to12(data.data.timings.Asr),
                        maghrib: convert24to12(data.data.timings.Maghrib),
                        isha: convert24to12(data.data.timings.Isha)
                    };
                    
                    this.state.currentHijriDate = `${data.data.date.hijri.day} ${data.data.date.hijri.month.en} ${data.data.date.hijri.year}`;
                    this.state.connectionStatus = 'connected';
                    this.state.lastUpdated = new Date();
                } catch(e) {
                    console.error(e);
                    this.state.connectionStatus = 'offline';
                }
                
                this.state.isLoading = false;
                this.render();
            },
            
            getNextPrayer() {
                const now = new Date();
                const currentHour = now.getHours();
                const currentMin = now.getMinutes();
                const currentTimeInMin = currentHour * 60 + currentMin;
                
                // Convert 12hr format back to minutes for comparison
                const timeToMinutes = (time12) => {
                    const match = time12.match(/(\d+):(\d+)\s*(AM|PM)/i);
                    if (!match) return 0;
                    let hours = parseInt(match[1]);
                    const minutes = parseInt(match[2]);
                    const period = match[3].toUpperCase();
                    
                    if (period === 'PM' && hours !== 12) hours += 12;
                    if (period === 'AM' && hours === 12) hours = 0;
                    
                    return hours * 60 + minutes;
                };
                
                const prayers = [
                    ['fajr', this.state.prayerTimes.fajr],
                    ['dhuhr', this.state.prayerTimes.dhuhr],
                    ['asr', this.state.prayerTimes.asr],
                    ['maghrib', this.state.prayerTimes.maghrib],
                    ['isha', this.state.prayerTimes.isha]
                ];
                
                for(let [prayer, time] of prayers) {
                    const prayerTimeInMin = timeToMinutes(time);
                    if(prayerTimeInMin > currentTimeInMin) {
                        // Calculate time remaining
                        const diffMin = prayerTimeInMin - currentTimeInMin;
                        const hours = Math.floor(diffMin / 60);
                        const minutes = diffMin % 60;
                        
                        let remaining = '';
                        if (hours > 0) {
                            remaining = `${hours} ساعة و ${minutes} دقيقة`;
                        } else {
                            remaining = `${minutes} دقيقة`;
                        }
                        
                        this.state.timeRemaining = remaining;
                        return {name: prayer, time: time};
                    }
                }
                
                // Next day's Fajr
                const fajrTimeInMin = timeToMinutes(this.state.prayerTimes.fajr);
                const diffMin = (24 * 60) - currentTimeInMin + fajrTimeInMin;
                const hours = Math.floor(diffMin / 60);
                const minutes = diffMin % 60;
                this.state.timeRemaining = `${hours} ساعة و ${minutes} دقيقة`;
                
                return {name: 'fajr', time: this.state.prayerTimes.fajr};
            },
            
            
            
            handleLogoUpload(event) {
                const file = event.target.files[0];
                if (file && file.type.startsWith('image/')) {
                    const reader = new FileReader();
                    reader.onload = (e) => {
                        this.state.customLogo = e.target.result;
                        localStorage.setItem('customLogo', e.target.result);
                        alert('✅ Logo uploaded successfully!');
                        this.render();
                    };
                    reader.readAsDataURL(file);
                } else {
                    alert('Please upload a valid image file (PNG, JPG, etc.)');
                }
            },
            
            shareApp() {
                const shareData = {
                    title: 'أبو داود - Islamic Prayer App',
                    text: 'Check out this beautiful Islamic prayer times app for Saida, Lebanon! Features: Daily prayer times, Adhan audio, Athkar, and more. Made by Abu Dawod.',
                    url: window.location.href
                };
                
                if (navigator.share) {
                    navigator.share(shareData)
                        .then(() => {
                            document.getElementById('shareStatus').textContent = '✅ Shared successfully!';
                            setTimeout(() => {
                                document.getElementById('shareStatus').textContent = '';
                            }, 3000);
                        })
                        .catch((error) => {
                            if (error.name !== 'AbortError') {
                                this.copyLink();
                            }
                        });
                } else {
                    this.copyLink();
                }
            },
            
            copyLink() {
                const link = window.location.href;
                
                if (navigator.clipboard && navigator.clipboard.writeText) {
                    navigator.clipboard.writeText(link)
                        .then(() => {
                            document.getElementById('shareStatus').textContent = '✅ Link copied to clipboard!';
                            setTimeout(() => {
                                document.getElementById('shareStatus').textContent = '';
                            }, 3000);
                        })
                        .catch(() => {
                            this.fallbackCopyLink(link);
                        });
                } else {
                    this.fallbackCopyLink(link);
                }
            },
            
            fallbackCopyLink(link) {
                const textArea = document.createElement('textarea');
                textArea.value = link;
                textArea.style.position = 'fixed';
                textArea.style.left = '-999999px';
                document.body.appendChild(textArea);
                textArea.focus();
                textArea.select();
                
                try {
                    document.execCommand('copy');
                    document.getElementById('shareStatus').textContent = '✅ Link copied! Share it with WhatsApp, Telegram, etc.';
                    setTimeout(() => {
                        document.getElementById('shareStatus').textContent = '';
                    }, 5000);
                } catch (err) {
                    document.getElementById('shareStatus').textContent = '⚠️ Please manually copy: ' + link;
                }
                
                document.body.removeChild(textArea);
            },
            
            handleAudioUpload(event) {
                const file = event.target.files[0];
                if (file && file.type.startsWith('audio/')) {
                    const audioUrl = URL.createObjectURL(file);
                    this.state.uploadedAudio = audioUrl;
                    alert('✅ Audio file uploaded successfully! Click Play to listen.');
                    this.render();
                } else {
                    alert('Please upload a valid audio file (MP3, WAV, etc.)');
                }
            },
            
            playAdhan() {
                if(this.state.isPlayingAdhan && this.state.currentAudio) {
                    this.state.currentAudio.pause();
                    this.state.currentAudio.currentTime = 0;
                    this.state.isPlayingAdhan = false;
                    this.state.currentAudio = null;
                    this.render();
                    return;
                }
                
                // Use uploaded audio, or selected muadhun URL
                let audioSource = this.state.uploadedAudio || this.muadhuns[this.state.selectedMuadhun].url;
                
                if (!audioSource) {
                    alert('Please upload an MP3 file first, or make sure the MP3 files are in the same folder as this HTML file.');
                    return;
                }
                
                const audio = new Audio(audioSource);
                
                audio.addEventListener('error', (e) => {
                    console.error('Audio error:', e);
                    alert('⚠️ Cannot load audio file.\n\n✅ INSTRUCTIONS:\n1. Save this HTML file in your Downloads folder\n2. Make sure your MP3 files are in the same folder\n3. Or use the "Upload Audio" button to select any MP3 file');
                    this.state.isPlayingAdhan = false;
                    this.render();
                });
                
                audio.play().then(() => {
                    console.log('✅ Audio playing!');
                    this.state.isPlayingAdhan = true;
                    this.state.currentAudio = audio;
                    this.render();
                }).catch((err) => {
                    console.error('Play error:', err);
                    if (err.name === 'NotAllowedError') {
                        alert('Browser blocked audio! Click anywhere on the page first, then try again.');
                    } else {
                        alert('⚠️ Cannot play audio.\n\n✅ SOLUTION: Use the "Upload Audio" button to select your MP3 file manually.');
                    }
                    this.state.isPlayingAdhan = false;
                    this.render();
                });
                
                audio.onended = () => {
                    this.state.isPlayingAdhan = false;
                    this.state.currentAudio = null;
                    this.render();
                };
            },
            
            async toggleNotifications() {
                if(!this.state.notificationsEnabled) {
                    if('Notification' in window) {
                        const perm = await Notification.requestPermission();
                        if(perm === 'granted') {
                            this.state.notificationsEnabled = true;
                            localStorage.setItem('notificationsEnabled', 'true');
                        }
                    }
                } else {
                    this.state.notificationsEnabled = false;
                    localStorage.setItem('notificationsEnabled', 'false');
                }
                this.render();
            },
            
            icon(name) {
                const icons = {
                    moon: '<path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path>',
                    bell: '<path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"></path><path d="M13.73 21a2 2 0 0 1-3.46 0"></path>',
                    clock: '<circle cx="12" cy="12" r="10"></circle><polyline points="12 6 12 12 16 14"></polyline>',
                    sun: '<circle cx="12" cy="12" r="5"></circle><line x1="12" y1="1" x2="12" y2="3"></line><line x1="12" y1="21" x2="12" y2="23"></line><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line><line x1="1" y1="12" x2="3" y2="12"></line><line x1="21" y1="12" x2="23" y2="12"></line><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line>',
                    book: '<path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20"></path><path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z"></path>',
                    heart: '<path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"></path>',
                    wifi: '<path d="M5 12.55a11 11 0 0 1 14.08 0"></path><path d="M1.42 9a16 16 0 0 1 21.16 0"></path><path d="M8.53 16.11a6 6 0 0 1 6.95 0"></path><line x1="12" y1="20" x2="12.01" y2="20"></line>',
                    wifioff: '<line x1="1" y1="1" x2="23" y2="23"></line><path d="M16.72 11.06A10.94 10.94 0 0 1 19 12.55"></path><path d="M5 12.55a10.94 10.94 0 0 1 5.17-2.39"></path><path d="M10.71 5.05A16 16 0 0 1 22.58 9"></path><path d="M1.42 9a15.91 15.91 0 0 1 4.7-2.88"></path><path d="M8.53 16.11a6 6 0 0 1 6.95 0"></path><line x1="12" y1="20" x2="12.01" y2="20"></line>',
                    refresh: '<polyline points="23 4 23 10 17 10"></polyline><polyline points="1 20 1 14 7 14"></polyline><path d="M3.51 9a9 9 0 0 1 14.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0 0 20.49 15"></path>',
                    pin: '<path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z"></path><circle cx="12" cy="10" r="3"></circle>'
                };
                return `<svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">${icons[name]}</svg>`;
            },
            
            
            translatePrayerName(prayer) {
                const names = {
                    fajr: 'الفجر',
                    sunrise: 'الشروق',
                    dhuhr: 'الظهر',
                    asr: 'العصر',
                    maghrib: 'المغرب',
                    isha: 'العشاء'
                };
                return names[prayer] || prayer;
            },
            
            render() {
                const s = this.state;
                const next = this.getNextPrayer();
                
                document.getElementById('app').innerHTML = `
                    <div class="min-h-screen">
                        <!-- App Header with Logo -->
                        <div class="bg-gradient-to-r from-green-600 to-blue-600 text-white p-6 shadow-lg">
                            <!-- App Logo -->
                            <div class="app-logo arabic-text">أد</div>
                            
                            <div class="flex items-center justify-between flex-wrap gap-4">
                                <div class="flex items-center space-x-3">
                                    <div>
                                        <h1 class="text-2xl font-bold arabic-text">أبو داود</h1>
                                        <p class="text-sm text-green-100">Prayer Times - Saida, Lebanon</p>
                                    </div>
                                </div>
                                <div class="text-right">
                                    <div class="text-2xl font-bold">${s.currentTime.toLocaleTimeString('en-US', {hour: 'numeric', minute: '2-digit', hour12: true})}</div>
                                    <div class="text-sm opacity-90">${s.currentTime.toLocaleDateString()}</div>
                                    ${s.currentHijriDate ? `<div class="text-xs opacity-75 mt-1 arabic-text">${s.currentHijriDate}</div>` : ''}
                                </div>
                            </div>
                            
                            <!-- Custom Logo Section -->
                            ${s.customLogo ? `
                                <div class="mt-6 text-center">
                                    <img src="${s.customLogo}" class="custom-logo-img mx-auto" style="max-height: 120px; border-radius: 15px; box-shadow: 0 5px 15px rgba(0,0,0,0.2);" alt="App Logo">
                                </div>
                            ` : ''}
                        </div>

                        <!-- Tabs -->
                        <div class="bg-white shadow-sm">
                            <div class="flex overflow-x-auto" dir="rtl">
                                ${[
                                    {id: 'prayer', label: 'الصلاة'},
                                    {id: 'adhan', label: 'الأذان'},
                                    {id: 'athkar', label: 'الأذكار'},
                                    {id: 'tawba', label: 'التوبة'},
                                    {id: 'profile', label: '👤 الملف'}
                                ].map(tab => `
                                    <button onclick="app.state.activeTab='${tab.id}'; app.render()" 
                                        class="tab-button flex-1 flex items-center justify-center space-x-2 py-4 px-6 border-b-2 transition-colors whitespace-nowrap ${
                                            s.activeTab === tab.id ? 'border-green-500 text-green-600 bg-green-50' : 'border-transparent text-gray-500 hover:text-gray-700'
                                        }">
                                        <span class="font-medium arabic-text">${tab.label}</span>
                                    </button>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Made by Abu Dawod Banner -->
                        <div class="p-6 pb-4">
                            <div class="bg-gradient-to-r from-green-600 to-blue-600 text-white p-4 rounded-xl shadow-lg text-center">
                                <p class="text-lg font-semibold arabic-text">صُنع بـ ❤️ من قبل أبو داود</p>
                                <p class="text-sm opacity-90">May Allah accept this work and make it beneficial for all Muslims</p>
                            </div>
                        </div>

                        <!-- Content -->
                        <div class="p-6 pt-0">
                            ${s.activeTab === 'prayer' ? this.renderPrayer(next) : ''}
                            ${s.activeTab === 'adhan' ? this.renderAdhan() : ''}
                            ${s.activeTab === 'athkar' ? this.renderAthkar() : ''}
                            ${s.activeTab === 'tawba' ? this.renderTawba() : ''}
                            ${s.activeTab === 'profile' ? this.renderProfile() : ''}
                        </div>
                    </div>
                `;
            },
            
            renderProfile() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <!-- Developer Profile Header -->
                        <div class="bg-gradient-to-r from-green-600 via-emerald-600 to-blue-600 text-white p-8 rounded-xl shadow-2xl">
                            <div class="text-center">
                                <!-- Profile Picture Upload Area -->
                                <div class="mb-6">
                                    <div class="w-32 h-32 mx-auto rounded-full bg-white bg-opacity-20 flex items-center justify-center cursor-pointer hover:bg-opacity-30 transition-all overflow-hidden border-4 border-white shadow-lg" onclick="document.getElementById('logoUpload').click()">
                                        ${s.customLogo ? `
                                            <img src="${s.customLogo}" class="w-full h-full object-cover" alt="Profile">
                                        ` : `
                                            <svg class="w-16 h-16 text-white opacity-75" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"></path>
                                            </svg>
                                        `}
                                    </div>
                                    <input 
                                        type="file" 
                                        id="logoUpload"
                                        accept="image/*" 
                                        onchange="app.handleLogoUpload(event)"
                                        style="display: none;"
                                    />
                                    <p class="text-sm mt-2 opacity-75">Click to upload profile photo</p>
                                </div>
                                
                                <h1 class="text-4xl font-bold mb-2 arabic-text">أبو داود</h1>
                                <p class="text-xl mb-2">Abu Dawod</p>
                                <p class="text-lg opacity-90">Islamic App Developer</p>
                                <div class="flex items-center justify-center gap-2 mt-4">
                                    <span class="px-4 py-1 bg-white bg-opacity-20 rounded-full text-sm">Saida, Lebanon 🇱🇧</span>
                                </div>
                            </div>
                        </div>

                        <!-- About This Project -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">📱 About This Application</h2>
                            <div class="space-y-4 text-gray-700">
                                <p class="leading-relaxed">
                                    <strong class="text-green-600">Islamic Prayer Times & Athkar App</strong> is a comprehensive web application designed to help Muslims stay connected with their daily prayers and remembrance of Allah.
                                </p>
                                <p class="leading-relaxed">
                                    This app provides accurate prayer times for Saida, Lebanon, beautiful Adhan audio, daily Athkar, and reminders for Tawba (repentance).
                                </p>
                            </div>
                        </div>

                        <!-- Features -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">✨ Key Features</h2>
                            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                                ${[
                                    { icon: '🕌', title: 'Daily Prayer Times', desc: 'Accurate times for Saida updated automatically' },
                                    { icon: '🔔', title: 'Prayer Notifications', desc: 'Get notified when it\'s time to pray' },
                                    { icon: '🎵', title: 'Beautiful Adhan', desc: 'Multiple Adhan styles with audio playback' },
                                    { icon: '📿', title: 'Daily Athkar', desc: 'Morning, evening, and general remembrances' },
                                    { icon: '💜', title: 'Tawba Reminders', desc: 'Encouragement for repentance and forgiveness' },
                                    { icon: '🌙', title: 'Hijri Calendar', desc: 'Display Islamic date alongside Gregorian' },
                                    { icon: '📱', title: 'Installable App', desc: 'Works like a native mobile application' },
                                    { icon: '🔄', title: 'Auto-Updates', desc: 'Prayer times update daily automatically' }
                                ].map(f => `
                                    <div class="flex items-start space-x-3 p-3 bg-green-50 rounded-lg">
                                        <span class="text-2xl">${f.icon}</span>
                                        <div>
                                            <h3 class="font-semibold text-gray-800">${f.title}</h3>
                                            <p class="text-sm text-gray-600">${f.desc}</p>
                                        </div>
                                    </div>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Technologies Used -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">🛠️ Technologies Used</h2>
                            <div class="flex flex-wrap gap-3">
                                ${['HTML5', 'CSS3', 'JavaScript', 'Tailwind CSS', 'AlAdhan API', 'Web Audio API', 'Local Storage', 'PWA'].map(tech => `
                                    <span class="px-4 py-2 bg-gradient-to-r from-green-100 to-blue-100 text-gray-800 rounded-full text-sm font-semibold border border-green-200">${tech}</span>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Statistics -->
                        <div class="bg-gradient-to-r from-purple-50 to-pink-50 p-6 rounded-xl shadow-lg border-2 border-purple-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">📊 App Statistics</h2>
                            <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
                                ${[
                                    { num: '5', label: 'Daily Prayers' },
                                    { num: '10+', label: 'Athkar Duas' },
                                    { num: '4', label: 'Adhan Styles' },
                                    { num: '100%', label: 'Free & Open' }
                                ].map(stat => `
                                    <div class="text-center p-4 bg-white rounded-lg shadow-sm">
                                        <div class="text-3xl font-bold text-green-600">${stat.num}</div>
                                        <div class="text-sm text-gray-600 mt-1">${stat.label}</div>
                                    </div>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Contact & Share -->
                        <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                            <h2 class="text-2xl font-bold mb-4 text-gray-800">🤝 Share This App</h2>
                            <p class="text-gray-700 mb-4">Help spread the benefit! Share this Islamic prayer app with your community:</p>
                            <div class="space-y-3">
                                <button onclick="app.shareApp()" 
                                    class="w-full px-6 py-3 bg-gradient-to-r from-green-500 to-blue-500 text-white rounded-lg font-semibold hover:from-green-600 hover:to-blue-600 transition-all shadow-md">
                                    📤 Share App
                                </button>
                                <button onclick="app.copyLink()" 
                                    class="w-full px-6 py-3 bg-gray-100 text-gray-800 rounded-lg font-semibold hover:bg-gray-200 transition-all">
                                    📋 Copy App Link
                                </button>
                                <div class="text-center text-sm text-gray-500 mt-2">
                                    <p id="shareStatus"></p>
                                </div>
                            </div>
                        </div>

                        <!-- Credits & License -->
                        <div class="bg-gradient-to-r from-green-600 to-blue-600 text-white p-6 rounded-xl shadow-lg text-center">
                            <h2 class="text-2xl font-bold mb-3 arabic-text">بارك الله فيك</h2>
                            <p class="text-lg mb-4">May Allah bless this work and make it a source of continuous reward (Sadaqah Jariyah)</p>
                            <div class="text-sm opacity-90">
                                <p>Developed with ❤️ by <strong class="arabic-text">أبو داود</strong></p>
                                <p class="mt-2">Version 1.0.0 | 2025</p>
                                <p class="mt-2">Free for all Muslims to use and benefit from</p>
                            </div>
                        </div>

                        ${s.customLogo ? `
                            <div class="bg-white p-4 rounded-lg shadow-sm border border-gray-200 text-center">
                                <button 
                                    onclick="app.state.customLogo = null; localStorage.removeItem('customLogo'); app.render();"
                                    class="px-6 py-2 bg-red-50 hover:bg-red-100 text-red-600 rounded-lg font-medium transition-colors">
                                    🗑️ Remove Profile Photo
                                </button>
                            </div>
                        ` : ''}
                    </div>
                `;
            },
            
            renderSettings() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-purple-500 to-indigo-500 text-white p-6 rounded-xl shadow-lg">
                            <h2 class="text-2xl font-bold mb-2">⚙️ Settings</h2>
                            <p class="text-lg opacity-90">Upload your program image</p>
                        </div>

                        <!-- Upload Photo Only -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-lg font-semibold mb-4 text-center">📸 Program Image</h3>
                            
                            <div class="custom-logo-container" onclick="document.getElementById('logoUpload').click()">
                                ${s.customLogo ? `
                                    <img src="${s.customLogo}" class="custom-logo-img" alt="Program Image">
                                ` : `
                                    <div class="text-center text-gray-400">
                                        <svg class="w-16 h-16 mx-auto mb-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"></path>
                                        </svg>
                                        <p class="text-sm font-semibold">Click to upload photo</p>
                                    </div>
                                `}
                            </div>
                            
                            <input 
                                type="file" 
                                id="logoUpload"
                                accept="image/*" 
                                onchange="app.handleLogoUpload(event)"
                                style="display: none;"
                            />
                            
                            ${s.customLogo ? `
                                <button 
                                    onclick="app.state.customLogo = null; localStorage.removeItem('customLogo'); app.render();"
                                    class="w-full mt-4 px-4 py-3 bg-red-50 hover:bg-red-100 text-red-600 rounded-lg font-semibold transition-colors">
                                    🗑️ Remove Photo
                                </button>
                            ` : ''}
                        </div>
                    </div>
                `;
            },
            
            renderPrayer(next) {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-green-500 to-blue-500 text-white p-6 rounded-xl shadow-lg transform transition-all hover:shadow-2xl">
                            <div class="text-center">
                                <h3 class="text-lg font-semibold opacity-90 mb-2">الصلاة القادمة</h3>
                                <h2 class="text-4xl font-bold mb-2 arabic-text">${this.translatePrayerName(next.name)}</h2>
                                <p class="text-2xl opacity-90 mb-3">${next.time}</p>
                                
                                <!-- Time Remaining -->
                                <div class="bg-white bg-opacity-20 rounded-lg p-4 mt-4">
                                    <p class="text-sm opacity-90 mb-1">الوقت المتبقي</p>
                                    <p class="text-3xl font-bold arabic-text">${s.timeRemaining}</p>
                                </div>
                            </div>
                        </div>

                        <div class="bg-white p-4 rounded-lg shadow-sm border border-gray-200">
                            <div class="flex items-center justify-between mb-3">
                                <div class="flex items-center space-x-3 space-x-reverse">
                                    ${this.icon('bell')}
                                    <span class="font-medium arabic-text">تنبيهات الصلاة</span>
                                </div>
                                <button onclick="app.toggleNotifications()" 
                                    class="relative inline-flex h-6 w-11 items-center rounded-full transition-colors ${s.notificationsEnabled ? 'bg-green-600' : 'bg-gray-200'}">
                                    <span class="inline-block h-4 w-4 transform rounded-full bg-white transition-transform ${s.notificationsEnabled ? 'translate-x-6' : 'translate-x-1'}"></span>
                                </button>
                            </div>
                            <div class="flex items-center justify-between text-sm pt-3 border-t border-gray-100">
                                <div class="flex items-center space-x-2 space-x-reverse">
                                    ${s.connectionStatus === 'connected' ? `${this.icon('wifi')}<span class="text-green-600 arabic-text">متصل بصيدا</span>` : 
                                      s.connectionStatus === 'connecting' ? `<div class="animate-spin">${this.icon('refresh')}</div><span class="text-blue-600 arabic-text">جاري الاتصال...</span>` :
                                      `${this.icon('wifioff')}<span class="text-red-600 arabic-text">غير متصل</span>`}
                                </div>
                                ${s.lastUpdated ? `<span class="text-gray-500 arabic-text">آخر تحديث: ${s.lastUpdated.toLocaleTimeString('ar-LB', {hour: 'numeric', minute: '2-digit', hour12: true})}</span>` : ''}
                            </div>
                            <button onclick="app.fetchPrayerTimes()" ${s.isLoading ? 'disabled' : ''}
                                class="w-full mt-3 px-4 py-2 bg-blue-50 hover:bg-blue-100 text-blue-600 rounded-lg font-medium transition-colors disabled:opacity-50 arabic-text">
                                ${s.isLoading ? '🔄 جاري التحديث...' : '🔄 تحديث أوقات الصلاة'}
                            </button>
                        </div>

                        <div class="bg-white rounded-lg shadow-sm border border-gray-200 overflow-hidden">
                            <h3 class="text-lg font-semibold p-4 border-b border-gray-200 bg-gray-50 arabic-text text-right">أوقات الصلاة اليوم</h3>
                            <div class="divide-y divide-gray-100">
                                ${Object.entries(s.prayerTimes).map(([p,t]) => `
                                    <div class="flex items-center justify-between p-4 hover:bg-gray-50 transition-colors" dir="rtl">
                                        <div class="flex items-center space-x-3 space-x-reverse">
                                            <div class="w-3 h-3 rounded-full ${p === next.name ? 'bg-green-500 animate-pulse' : 'bg-gray-300'}"></div>
                                            <span class="font-medium text-gray-700 arabic-text text-lg">${this.translatePrayerName(p)}</span>
                                        </div>
                                        <span class="text-lg font-semibold text-gray-900">${t}</span>
                                    </div>
                                `).join('')}
                            </div>
                        </div>
                    </div>
                `;
            },
            
            renderAdhan() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-amber-500 to-orange-500 text-white p-6 rounded-xl shadow-lg">
                            <div class="flex items-center space-x-3 mb-4">
                                ${this.icon('sun')}
                                <h2 class="text-2xl font-bold">Adhan Player</h2>
                            </div>
                            <p class="text-lg opacity-90">Listen to the beautiful call to prayer</p>
                        </div>

                        <!-- Instructions -->
                        <div class="bg-blue-50 p-4 rounded-lg border-2 border-blue-200">
                            <h3 class="font-bold text-blue-800 mb-2">📋 Instructions:</h3>
                            <ol class="text-sm text-blue-700 space-y-1 list-decimal list-inside">
                                <li>Save this HTML file in your <strong>Downloads folder</strong></li>
                                <li>Your MP3 files should be in the <strong>same folder</strong></li>
                                <li>Or click "Choose File" below to upload any MP3</li>
                            </ol>
                        </div>

                        <!-- Upload Audio -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-lg font-semibold mb-4">📁 Upload Your Adhan MP3</h3>
                            <input
                                type="file"
                                accept="audio/*"
                                onchange="app.handleAudioUpload(event)"
                                class="block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-green-50 file:text-green-700 hover:file:bg-green-100 cursor-pointer"
                            />
                            ${s.uploadedAudio ? '<p class="text-green-600 text-sm mt-2">✅ Audio uploaded! Ready to play.</p>' : ''}
                        </div>

                        <!-- Choose Adhan -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-lg font-semibold mb-4">Choose Adhan Source</h3>
                            <div class="space-y-3">
                                ${Object.entries(this.muadhuns).map(([k,m]) => `
                                    <div onclick="app.state.selectedMuadhun='${k}'; app.render()" 
                                        class="p-4 rounded-lg border-2 cursor-pointer transition-all transform hover:scale-102 ${
                                            s.selectedMuadhun === k ? 'border-amber-500 bg-amber-50 shadow-md' : 'border-gray-200 hover:border-gray-300'
                                        }">
                                        <div class="flex items-center justify-between">
                                            <div>
                                                <h4 class="font-semibold text-gray-800">${m.name}</h4>
                                                <p class="text-sm text-gray-600">${m.desc}</p>
                                            </div>
                                            <div class="w-4 h-4 rounded-full border-2 ${s.selectedMuadhun === k ? 'border-amber-500 bg-amber-500' : 'border-gray-300'}"></div>
                                        </div>
                                    </div>
                                `).join('')}
                            </div>
                        </div>

                        <!-- Play Button -->
                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <div class="text-center space-y-4">
                                <h3 class="text-lg font-semibold">${this.muadhuns[s.selectedMuadhun].name}</h3>
                                <button onclick="app.playAdhan()" 
                                    class="px-8 py-4 rounded-full text-white font-semibold text-lg transition-all transform hover:scale-105 shadow-lg ${
                                        s.isPlayingAdhan ? 'bg-red-500 hover:bg-red-600 animate-pulse' : 'bg-gradient-to-r from-green-500 to-blue-500 hover:from-green-600 hover:to-blue-600'
                                    }">
                                    ${s.isPlayingAdhan ? '⏸️ Stop Adhan' : '▶️ Play Adhan'}
                                </button>
                                ${s.isPlayingAdhan ? `
                                    <div class="bg-green-50 p-4 rounded-lg border border-green-200 animate-pulse">
                                        <p class="text-green-700 font-medium">🎵 Playing Adhan...</p>
                                        <p class="text-sm text-gray-600 mt-1">${s.uploadedAudio ? 'Your uploaded audio' : this.muadhuns[s.selectedMuadhun].name}</p>
                                    </div>
                                ` : ''}
                                <p class="text-sm text-gray-500 mt-2">🔊 Make sure your volume is on</p>
                            </div>
                        </div>
                    </div>
                `;
            },
            
            renderAthkar() {
                const s = this.state;
                return `
                    <div class="space-y-6">
                        <div class="bg-white p-4 rounded-lg shadow-sm border border-gray-200">
                            <div class="flex space-x-2 overflow-x-auto">
                                ${Object.keys(this.athkar).map(cat => `
                                    <button onclick="app.state.selectedAthkar='${cat}'; app.render()" 
                                        class="px-4 py-2 rounded-lg font-medium capitalize transition-all whitespace-nowrap ${
                                            s.selectedAthkar === cat ? 'bg-green-500 text-white shadow-md' : 'bg-gray-100 text-gray-700 hover:bg-gray-200'
                                        }">
                                        ${cat}
                                    </button>
                                `).join('')}
                            </div>
                        </div>

                        ${this.athkar[s.selectedAthkar].map(d => `
                            <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200 transform transition-all hover:shadow-lg">
                                <div class="text-right mb-4">
                                    <p class="text-2xl text-green-700 leading-relaxed">${d.ar}</p>
                                </div>
                                <div class="space-y-2">
                                    <p class="text-gray-600 italic text-sm">${d.tr}</p>
                                    <p class="text-gray-800 font-medium">${d.en}</p>
                                </div>
                            </div>
                        `).join('')}
                    </div>
                `;
            },
            
            renderTawba() {
                return `
                    <div class="space-y-6">
                        <div class="bg-gradient-to-r from-purple-500 to-pink-500 text-white p-6 rounded-xl shadow-lg">
                            <div class="flex items-center space-x-3 space-x-reverse mb-4" dir="rtl">
                                ${this.icon('heart')}
                                <h2 class="text-2xl font-bold arabic-text">التوبة - الاستغفار</h2>
                            </div>
                            <p class="text-lg opacity-90 arabic-text text-right leading-relaxed">
                                "وَمَن تَابَ بَعْدَ ظُلْمِهِ وَأَصْلَحَ فَإِنَّ اللَّهَ يَتُوبُ عَلَيْهِ ۗ إِنَّ اللَّهَ غَفُورٌ رَّحِيمٌ" - سورة المائدة 39
                            </p>
                        </div>

                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-xl font-semibold mb-4 text-center arabic-text">الاستغفار اليومي</h3>
                            <div class="text-center">
                                <div class="text-4xl font-bold text-green-600 mb-4 arabic-text leading-relaxed">
                                    أَسْتَغْفِرُ اللَّهَ الْعَظِيمَ
                                </div>
                                <p class="text-gray-600 mb-2 italic">Astaghfiru Allah al-Adheem</p>
                                <p class="text-sm text-gray-500 mb-4 arabic-text">أستغفر الله العظيم وأتوب إليه</p>
                            </div>
                        </div>

                        <div class="space-y-4">
                            <h3 class="text-xl font-semibold text-gray-700 arabic-text text-right">أدعية التوبة والاستغفار</h3>
                            
                            <div class="bg-gradient-to-r from-blue-50 to-purple-50 p-6 rounded-lg border-l-4 border-purple-400">
                                <p class="text-2xl text-gray-800 arabic-text text-right leading-loose mb-3">
                                    اللَّهُمَّ إِنَّكَ عَفُوٌّ كَرِيمٌ تُحِبُّ الْعَفْوَ فَاعْفُ عَنَّا
                                </p>
                                <p class="text-sm text-gray-600 italic">Allahumma innaka 'afuwwun karimun tuhibbul-'afwa fa'fu 'anna</p>
                                <p class="text-sm text-gray-700 mt-2 arabic-text text-right">
                                    اللهم إنك عفو كريم تحب العفو فاعف عنا
                                </p>
                            </div>

                            <div class="bg-gradient-to-r from-green-50 to-blue-50 p-6 rounded-lg border-l-4 border-green-400">
                                <p class="text-2xl text-gray-800 arabic-text text-right leading-loose mb-3">
                                    رَبَّنَا آتِنَا فِي الدُّنْيَا حَسَنَةً وَفِي الْآخِرَةِ حَسَنَةً وَقِنَا عَذَابَ النَّارِ
                                </p>
                                <p class="text-sm text-gray-600 italic">Rabbana atina fid-dunya hasanatan wa fil-akhirati hasanatan wa qina 'adhaban-nar</p>
                                <p class="text-sm text-gray-700 mt-2 arabic-text text-right">
                                    اللهم آتنا في الدنيا حسنة وفي الآخرة حسنة وقنا عذاب النار
                                </p>
                            </div>

                            <div class="bg-gradient-to-r from-purple-50 to-pink-50 p-6 rounded-lg border-l-4 border-pink-400">
                                <p class="text-2xl text-gray-800 arabic-text text-right leading-loose mb-3">
                                    اللَّهُمَّ آتِ نَفْسِي تَقْوَاهَا وَزَكِّهَا أَنتَ خَيْرُ مَن زَكَّاهَا أَنتَ وَلِيُّهَا وَمَوْلَاهَا
                                </p>
                                <p class="text-sm text-gray-600 italic">Allahumma ati nafsi taqwaha wa zakkiha anta khayru man zakkaha anta waliyyuha wa mawlaha</p>
                                <p class="text-sm text-gray-700 mt-2 arabic-text text-right">
                                    اللهم آت نفسي تقواها وزكها أنت خير من زكاها أنت وليها ومولاها
                                </p>
                            </div>
                        </div>

                        <div class="space-y-4">
                            <h3 class="text-xl font-semibold text-gray-700 arabic-text text-right">تذكيرات يومية</h3>
                            ${this.tawbaReminders.map(r => `
                                <div class="bg-gradient-to-r from-blue-50 to-purple-50 p-4 rounded-lg border-r-4 border-purple-400 transform transition-all hover:shadow-md" dir="rtl">
                                    <p class="text-gray-700 arabic-text text-right">${r}</p>
                                </div>
                            `).join('')}
                        </div>

                        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200">
                            <h3 class="text-xl font-semibold mb-4 arabic-text text-right">دعاء الاستغفار</h3>
                            <div class="border-r-4 border-green-400 pr-4" dir="rtl">
                                <p class="text-2xl text-green-700 mb-3 text-right leading-loose arabic-text">
                                    رَبَّنَا ظَلَمْنَا أَنفُسَنَا وَإِن لَّمْ تَغْفِرْ لَنَا وَتَرْحَمْنَا لَنَكُونَنَّ مِنَ الْخَاسِرِينَ
                                </p>
                                <p class="text-sm text-gray-600 italic mb-1">
                                    Rabbana zalamna anfusana wa in lam taghfir lana wa tarhamna lanakoonanna minal-khasireen
                                </p>
                                <p class="text-sm text-gray-700 arabic-text text-right">
                                    "رَبَّنَا ظَلَمْنَا أَنفُسَنَا وَإِن لَّمْ تَغْفِرْ لَنَا وَتَرْحَمْنَا لَنَكُونَنَّ مِنَ الْخَاسِرِينَ"
                                </p>
                            </div>
                        </div>
                    </div>
                `;
            },
            
            init() {
                this.fetchPrayerTimes();
                setInterval(() => {
                    this.state.currentTime = new Date();
                    this.render();
                }, 1000);
                setInterval(() => this.fetchPrayerTimes(), 21600000);
                setInterval(() => {
                    const now = new Date();
                    if(now.getHours() === 0 && now.getMinutes() === 0) {
                        this.fetchPrayerTimes();
                    }
                }, 60000);
                this.render();
            }
        };
        
        window.app = app;
        app.init();
    </script>
</body>
</html>
\
