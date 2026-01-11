<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>نظام تقييم مشاريع التخرج | النسخة المطورة</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;500;700&display=swap" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <style>
        body { font-family: 'Tajawal', sans-serif; background-color: #f1f5f9; }
        .score-input { border: 2px solid #e2e8f0; transition: all 0.2s; text-align: center; font-weight: 700; font-size: 1.1rem; }
        .score-input:focus { border-color: #4f46e5; outline: none; background-color: #fffbeb; }
        .lang-en { display: block; font-size: 0.7rem; opacity: 0.6; font-weight: normal; margin-top: 2px; }
        @media print { .no-print { display: none; } body { background: white; padding: 0; } .container { box-shadow: none; border: none; width: 100%; max-width: 100%; } .student-card { border: 1px solid #eee; break-inside: avoid; } }
    </style>
</head>
<body class="p-4 md:p-8">

    <div id="app" class="max-w-6xl mx-auto space-y-6">
        
        <!-- Selection Screen -->
        <div id="roleSelection" class="bg-white p-10 rounded-[2rem] shadow-2xl text-center no-print border border-gray-100">
            <h2 class="text-3xl font-black mb-2 text-slate-800">بوابة التقييم الرقمية</h2>
            <p class="text-slate-500 mb-10">اختر نوع النموذج لبدء رصد العلامات</p>
            
            <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
                <button onclick="setRole('supervisor')" class="group p-10 bg-white border-4 border-indigo-600 rounded-[2rem] hover:bg-indigo-600 hover:text-white transition-all duration-300 shadow-xl">
                    <div class="text-5xl mb-4 group-hover:scale-110 transition-transform">📋</div>
                    <div class="text-2xl font-black">نموذج المشرف</div>
                    <div class="text-sm opacity-70 mt-1">Supervisor's Model</div>
                </button>
                
                <button onclick="setRole('examiner')" class="group p-10 bg-white border-4 border-emerald-600 rounded-[2rem] hover:bg-emerald-600 hover:text-white transition-all duration-300 shadow-xl">
                    <div class="text-5xl mb-4 group-hover:scale-110 transition-transform">🎓</div>
                    <div class="text-2xl font-black">نموذج المناقش</div>
                    <div class="text-sm opacity-70 mt-1">Examiner's Model</div>
                </button>
            </div>
        </div>

        <!-- Main Form Container -->
        <div id="mainContainer" class="hidden bg-white shadow-2xl rounded-[2.5rem] overflow-hidden container border border-gray-100">
            
            <!-- Header -->
            <div id="formHeader" class="p-10 text-white text-center relative">
                <button onclick="resetRole()" class="absolute top-6 left-6 bg-white/20 hover:bg-white/40 px-4 py-2 rounded-full text-xs no-print transition-all">
                    تغيير النموذج / Switch
                </button>
                <h1 id="headerTitle" class="text-4xl font-black mb-1"></h1>
                <p id="headerSub" class="text-lg opacity-90 font-medium"></p>
            </div>

            <form id="evaluationForm" class="p-8 md:p-12 space-y-10">
                
                <!-- Info Section -->
                <div id="infoSection" class="grid grid-cols-1 md:grid-cols-3 gap-8 pb-8 border-b-2 border-slate-50">
                    <div class="space-y-1">
                        <label class="block font-bold text-slate-700 text-sm">اختر المشروع <span class="lang-en">Select Project</span></label>
                        <select id="projectSelect" class="w-full p-2 bg-white border border-slate-200 rounded-lg outline-none font-medium" onchange="loadProjectData()">
                            <option value="">-- اختر المشروع --</option>
                            <option value="p1">نظام إدارة المستودعات الذكي</option>
                            <option value="p2">تطبيق التجارة الإلكترونية المتقدم</option>
                            <option value="p3">نظام الحماية باستخدام الذكاء الاصطناعي</option>
                            <option value="custom">مشروع آخر (يدوي)...</option>
                        </select>
                        <input type="text" id="projectTitle" class="w-full p-2 mt-2 bg-gray-50 border border-slate-200 rounded-lg hidden" placeholder="أدخل اسم المشروع">
                    </div>
                    <div id="dynamicFields" class="contents"></div>
                </div>

                <!-- Sync Marks Button (Only for Supervisor) -->
                <div id="syncSection" class="hidden no-print bg-amber-50 p-4 rounded-xl border border-amber-200 flex items-center justify-between">
                    <p class="text-sm text-amber-800 font-bold">💡 دمج علامات (الكتاب والعملي) لجميع الطلاب؟</p>
                    <button type="button" onclick="syncSharedMarks()" class="bg-amber-500 text-white px-4 py-2 rounded-lg text-xs font-bold hover:bg-amber-600 transition-all">تفعيل الدمج / Sync Marks</button>
                </div>

                <!-- Students Section -->
                <div class="grid grid-cols-1 lg:grid-cols-3 gap-8" id="studentsWrapper"></div>

                <!-- Actions -->
                <div class="pt-8 flex flex-col md:flex-row items-center justify-between gap-6 border-t-2 border-slate-50">
                    <div class="no-print flex flex-wrap justify-center gap-3">
                        <button type="button" onclick="exportToExcel()" class="bg-emerald-600 text-white px-6 py-3 rounded-xl font-bold hover:bg-emerald-700 shadow-lg flex items-center gap-2">
                            <span>ملف Excel</span>
                        </button>
                        <button type="button" onclick="shareWhatsApp()" class="bg-green-500 text-white px-6 py-3 rounded-xl font-bold hover:bg-green-600 shadow-lg flex items-center gap-2">
                            <span>WhatsApp</span>
                        </button>
                        <button type="button" onclick="window.print()" class="bg-slate-800 text-white px-6 py-3 rounded-xl font-bold hover:bg-slate-700 shadow-lg flex items-center gap-2">
                            <span>طباعة PDF</span>
                        </button>
                    </div>
                </div>
            </form>
        </div>
    </div>

    <!-- Student Template -->
    <template id="studentTemplate">
        <div class="student-card bg-slate-50/50 border-2 border-slate-100 rounded-[2rem] p-6 flex flex-col h-full transition-all">
            <div class="mb-4">
                <label class="block text-[10px] font-black text-slate-400 mb-1 uppercase">اختر الطالب / Student</label>
                <select class="student-name-select w-full bg-white border-2 border-slate-100 p-2 rounded-lg font-bold text-slate-700 focus:border-indigo-500 outline-none">
                    <option value="">-- اختر الطالب --</option>
                    <option>جمانة سعادة</option>
                    <option>شهد فقوسة</option>
                    <option>تامر غيث</option>
                    <option>منتصر اللجعبة</option>
                    <option>محمد خشان</option>
                    <option>رهف الجلاني </option>
                    <option>احمد البكري</option>
                    <option>احمد شكارنة </option>
                    <option>ضياء ابو عمر</option>
                    <option>اماتي ابو زنيد</option>
                    <option>شهد السعده</option>
                    <option>ضياء مشعل</option>
                    <option value="custom">اسم مخصص...</option>
                </select>
                <input type="text" class="student-name-input w-full bg-white border-2 border-slate-100 p-2 rounded-lg font-bold text-slate-700 focus:border-indigo-500 outline-none mt-2 hidden" placeholder="أدخل اسم الطالب">
            </div>
            <div class="criteria-list space-y-4 flex-grow"></div>
            <div class="mt-6 pt-4 border-t-2 border-dashed border-slate-200 flex justify-between items-center">
                <div>
                    <span class="text-[10px] font-black text-slate-400 block uppercase">Total</span>
                    <span class="text-3xl font-black text-slate-800 student-total-display">0</span>
                </div>
                <span class="student-result-text font-bold text-[10px] px-2 py-1 bg-slate-200 rounded-full italic">N/A</span>
            </div>
        </div>
    </template>

    <script>
        let currentRole = '';
        let isSyncing = false;
        
        const config = {
            supervisor: {
                title: "نموذج تقييم المشرف", sub: "Supervisor Evaluation", color: "bg-indigo-700",
                fields: [
                    { id: 'supervisorName', label: 'اسم المشرف', en: 'Supervisor Name' },
                    { id: 'date', label: 'التاريخ', en: 'Date', type: 'date' }
                ],
                criteria: [
                    { id: 'book', label: 'الكتاب (التوثيق)', max: 25, shared: true },
                    { id: 'practical', label: 'الجانب العملي', max: 35, shared: true },
                    { id: 'reviews', label: 'المراجعات الدورية', max: 20, shared: false },
                    { id: 'teamwork', label: 'تعاون الفريق', max: 20, shared: false }
                ]
            },
            examiner: {
                title: "نموذج تقييم المناقش", sub: "Examiner Evaluation", color: "bg-emerald-700",
                fields: [
                    { id: 'supervisorName', label: 'اسم المشرف', en: 'Supervisor' },
                    { id: 'examinerName', label: 'اسم المناقش', en: 'Examiner' },
                    { id: 'date', label: 'التاريخ', en: 'Date', type: 'date' }
                ],
                criteria: [
                    { id: 'report', label: 'جودة التقرير', max: 25 },
                    { id: 'demo', label: 'العرض العملي', max: 30 },
                    { id: 'presentation', label: 'مهارات العرض', max: 20 },
                    { id: 'scientific', label: 'التمكن العلمي', max: 25 }
                ]
            }
        };

        function loadProjectData() {
            const select = document.getElementById('projectSelect');
            const input = document.getElementById('projectTitle');
            if(select.value === 'custom') {
                input.classList.remove('hidden');
            } else {
                input.classList.add('hidden');
                input.value = select.options[select.selectedIndex].text;
            }
        }

        function setRole(role) {
            currentRole = role;
            const data = config[role];
            document.getElementById('roleSelection').classList.add('hidden');
            document.getElementById('mainContainer').classList.remove('hidden');
            document.getElementById('formHeader').className = `p-10 text-white text-center relative ${data.color}`;
            document.getElementById('headerTitle').innerText = data.title;
            document.getElementById('headerSub').innerText = data.sub;

            if (role === 'supervisor') document.getElementById('syncSection').classList.remove('hidden');

            const dynamicFields = document.getElementById('dynamicFields');
            dynamicFields.innerHTML = '';
            data.fields.forEach(f => {
                const div = document.createElement('div');
                div.innerHTML = `<label class="block font-bold text-slate-700 text-sm">${f.label}</label>
                                 <input type="${f.type || 'text'}" id="${f.id}" class="w-full p-2 bg-white border border-slate-200 rounded-lg outline-none font-medium">`;
                dynamicFields.appendChild(div);
            });

            const wrapper = document.getElementById('studentsWrapper');
            const template = document.getElementById('studentTemplate');
            wrapper.innerHTML = '';
            for (let i = 1; i <= 3; i++) {
                const clone = template.content.cloneNode(true);
                const card = clone.querySelector('.student-card');
                const critList = clone.querySelector('.criteria-list');
                const nameSelect = clone.querySelector('.student-name-select');
                const nameInput = clone.querySelector('.student-name-input');

                nameSelect.onchange = () => {
                    if(nameSelect.value === 'custom') nameInput.classList.remove('hidden');
                    else { nameInput.classList.add('hidden'); nameInput.value = nameSelect.value; }
                };

                data.criteria.forEach(c => {
                    const row = document.createElement('div');
                    row.innerHTML = `<div class="flex justify-between text-[10px] font-bold text-slate-500 mb-1"><span>${c.label}</span><span>Max: ${c.max}</span></div>
                                     <input type="number" min="0" max="${c.max}" value="0" class="score-input w-full p-1 rounded-lg border focus:ring-2 focus:ring-indigo-100" data-id="${c.id}" data-shared="${c.shared || false}">`;
                    
                    const input = row.querySelector('input');
                    input.addEventListener('input', (e) => {
                        if (e.target.value > c.max) e.target.value = c.max;
                        if (isSyncing && c.shared) {
                            applySync(c.id, e.target.value);
                        }
                        updateTotal(card);
                    });
                    critList.appendChild(row);
                });
                wrapper.appendChild(clone);
            }
        }

        function syncSharedMarks() {
            isSyncing = !isSyncing;
            const btn = event.target;
            btn.innerText = isSyncing ? "إيقاف الدمج / Unsync" : "تفعيل الدمج / Sync Marks";
            btn.className = isSyncing ? "bg-red-500 text-white px-4 py-2 rounded-lg text-xs font-bold hover:bg-red-600 transition-all" : "bg-amber-500 text-white px-4 py-2 rounded-lg text-xs font-bold hover:bg-amber-600 transition-all";
        }

        function applySync(criteriaId, value) {
            document.querySelectorAll(`.score-input[data-id="${criteriaId}"]`).forEach(input => {
                input.value = value;
                updateTotal(input.closest('.student-card'));
            });
        }

        function updateTotal(card) {
            const inputs = card.querySelectorAll('.score-input');
            let total = 0;
            inputs.forEach(i => total += (parseInt(i.value) || 0));
            card.querySelector('.student-total-display').innerText = total;
            const res = card.querySelector('.student-result-text');
            if (total >= 90) { res.innerText = "امتياز"; res.className = "student-result-text font-bold text-[10px] px-2 py-1 bg-indigo-100 rounded-full text-indigo-700"; }
            else if (total >= 50) { res.innerText = "ناجح"; res.className = "student-result-text font-bold text-[10px] px-2 py-1 bg-emerald-100 rounded-full text-emerald-700"; }
            else { res.innerText = "راسب"; res.className = "student-result-text font-bold text-[10px] px-2 py-1 bg-rose-100 rounded-full text-rose-700"; }
        }

        function exportToExcel() {
            const data = config[currentRole];
            const project = document.getElementById('projectTitle').value || 'Untitled';
            const excelData = [
                ["تقرير تقييم مشروع تخرج - " + data.title],
                ["عنوان المشروع:", project],
                ["المشرف:", document.getElementById('supervisorName').value],
                ["تاريخ التقييم:", document.getElementById('date').value],
                [],
                ["اسم الطالب", "المجموع (100)", "النتيجة"]
            ];
            document.querySelectorAll('.student-card').forEach(card => {
                const name = card.querySelector('.student-name-input').value;
                const total = card.querySelector('.student-total-display').innerText;
                const result = card.querySelector('.student-result-text').innerText;
                if (name) excelData.push([name, total, result]);
            });
            const ws = XLSX.utils.aoa_to_sheet(excelData);
            const wb = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(wb, ws, "Evaluation");
            XLSX.writeFile(wb, `Graduation_Evaluation_${project}.xlsx`);
        }

        function shareWhatsApp() {
            const data = config[currentRole];
            const project = document.getElementById('projectTitle').value || 'مشروع غير مسمى';
            let msg = `*${data.title}*%0A*المشروع:* ${project}%0A------------------%0A`;
            document.querySelectorAll('.student-card').forEach(card => {
                const name = card.querySelector('.student-name-input').value;
                const total = card.querySelector('.student-total-display').innerText;
                const result = card.querySelector('.student-result-text').innerText;
                if (name) msg += `👤 *${name}*: ${total}/100 (${result})%0A`;
            });
            window.open(`https://wa.me/?text=${msg}`, '_blank');
        }

        function resetRole() { if(confirm("سيتم حذف البيانات الحالية، هل أنت متأكد؟")) location.reload(); }
    </script>
</body>
</html>
