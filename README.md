<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>نظام تقييم مشاريع التخرج - علامات منفصلة | Individual Team Grading</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;500;700&display=swap" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, serverTimestamp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const firebaseConfig = JSON.parse(__firebase_config);
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'grad-eval-app';

        async function initAuth() {
            try { await signInAnonymously(auth); } catch (e) { console.error(e); }
        }
        initAuth();

        window.saveToCloud = async function() {
            const user = auth.currentUser;
            if (!user) { alert("جاري الاتصال... / Connecting..."); return; }
            const data = getFormData();
            try {
                const btn = document.getElementById('saveCloudBtn');
                btn.disabled = true; btn.innerText = "جاري الحفظ... / Saving...";
                await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'evaluations'), {
                    ...data,
                    timestamp: serverTimestamp(),
                    userId: user.uid
                });
                alert("تم الحفظ بنجاح! / Saved Successfully!");
            } catch (e) { alert("خطأ في الحفظ / Save Error"); }
            finally { 
                const btn = document.getElementById('saveCloudBtn');
                btn.disabled = false; btn.innerText = "حفظ سحابي / Cloud Save";
            }
        };
    </script>
    <style>
        body { font-family: 'Tajawal', sans-serif; background-color: #f1f5f9; }
        .lang-en { display: block; font-size: 0.75rem; opacity: 0.7; font-weight: normal; }
        .score-card { transition: all 0.3s ease; }
        .score-card:focus-within { border-color: #4f46e5; transform: translateY(-2px); }
        @media print { .no-print { display: none; } body { background: white; } }
    </style>
</head>
<body class="p-4 md:p-8">

    <div class="max-w-6xl mx-auto bg-white shadow-2xl rounded-3xl overflow-hidden container">
        <!-- Header -->
        <div class="bg-indigo-900 p-8 text-white text-center border-b-4 border-yellow-500">
            <h1 class="text-3xl font-bold">تقييم مشروع التخرج (فريق)</h1>
            <p class="text-lg opacity-80">Graduation Project Evaluation (Team)</p>
        </div>

        <form id="evaluationForm" class="p-6 md:p-10 space-y-10">
            <!-- Project Info -->
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6 pb-8 border-b">
                <div class="space-y-2">
                    <label class="font-bold">عنوان المشروع <span class="lang-en">Project Title</span></label>
                    <input type="text" id="projectTitle" class="w-full p-3 bg-gray-50 border rounded-xl" placeholder="عنوان البحث">
                </div>
                <div class="space-y-2">
                    <label class="font-bold">المشرف <span class="lang-en">Supervisor</span></label>
                    <input type="text" id="supervisor" class="w-full p-3 bg-gray-50 border rounded-xl">
                </div>
                <div class="space-y-2">
                    <label class="font-bold">المناقش <span class="lang-en">Examiner</span></label>
                    <input type="text" id="examiner" class="w-full p-3 bg-gray-50 border rounded-xl">
                </div>
                <div class="space-y-2">
                    <label class="font-bold">التاريخ <span class="lang-en">Date</span></label>
                    <input type="date" id="evalDate" class="w-full p-3 bg-gray-50 border rounded-xl">
                </div>
            </div>

            <!-- Students Section -->
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
                <!-- Student 1 -->
                <div class="bg-blue-50/50 p-6 rounded-3xl border border-blue-100 space-y-4">
                    <div class="flex justify-between items-center">
                        <span class="bg-blue-600 text-white px-3 py-1 rounded-full text-xs font-bold">طالب 1 / S1</span>
                    </div>
                    <input type="text" id="name1" placeholder="اسم الطالب الأول" class="w-full p-3 border rounded-xl mb-4 font-bold">
                    
                    <div class="space-y-3">
                        <div>
                            <label class="text-xs font-bold text-gray-500">تقرير (30) <span class="lang-en">Report</span></label>
                            <input type="number" id="s1_r" max="30" value="0" oninput="calc(1)" class="w-full p-2 border rounded-lg text-center font-black text-blue-700">
                        </div>
                        <div>
                            <label class="text-xs font-bold text-gray-500">عملي (40) <span class="lang-en">Practical</span></label>
                            <input type="number" id="s1_p" max="40" value="0" oninput="calc(1)" class="w-full p-2 border rounded-lg text-center font-black text-blue-700">
                        </div>
                        <div>
                            <label class="text-xs font-bold text-gray-500">مناقشة (30) <span class="lang-en">Presentation</span></label>
                            <input type="number" id="s1_d" max="30" value="0" oninput="calc(1)" class="w-full p-2 border rounded-lg text-center font-black text-blue-700">
                        </div>
                    </div>
                    <div class="pt-4 border-t border-blue-200 text-center">
                        <span class="text-xs text-gray-400 block">المجموع / Total</span>
                        <span id="total1" class="text-3xl font-black text-blue-800">0</span>
                    </div>
                </div>

                <!-- Student 2 -->
                <div class="bg-indigo-50/50 p-6 rounded-3xl border border-indigo-100 space-y-4">
                    <div class="flex justify-between items-center">
                        <span class="bg-indigo-600 text-white px-3 py-1 rounded-full text-xs font-bold">طالب 2 / S2</span>
                    </div>
                    <input type="text" id="name2" placeholder="اسم الطالب الثاني" class="w-full p-3 border rounded-xl mb-4 font-bold">
                    
                    <div class="space-y-3">
                        <div>
                            <label class="text-xs font-bold text-gray-500">تقرير (30) <span class="lang-en">Report</span></label>
                            <input type="number" id="s2_r" max="30" value="0" oninput="calc(2)" class="w-full p-2 border rounded-lg text-center font-black text-indigo-700">
                        </div>
                        <div>
                            <label class="text-xs font-bold text-gray-500">عملي (40) <span class="lang-en">Practical</span></label>
                            <input type="number" id="s2_p" max="40" value="0" oninput="calc(2)" class="w-full p-2 border rounded-lg text-center font-black text-indigo-700">
                        </div>
                        <div>
                            <label class="text-xs font-bold text-gray-500">مناقشة (30) <span class="lang-en">Presentation</span></label>
                            <input type="number" id="s2_d" max="30" value="0" oninput="calc(2)" class="w-full p-2 border rounded-lg text-center font-black text-indigo-700">
                        </div>
                    </div>
                    <div class="pt-4 border-t border-indigo-200 text-center">
                        <span class="text-xs text-gray-400 block">المجموع / Total</span>
                        <span id="total2" class="text-3xl font-black text-indigo-800">0</span>
                    </div>
                </div>

                <!-- Student 3 -->
                <div class="bg-purple-50/50 p-6 rounded-3xl border border-purple-100 space-y-4">
                    <div class="flex justify-between items-center">
                        <span class="bg-purple-600 text-white px-3 py-1 rounded-full text-xs font-bold">طالب 3 / S3</span>
                    </div>
                    <input type="text" id="name3" placeholder="اسم الطالب الثالث" class="w-full p-3 border rounded-xl mb-4 font-bold">
                    
                    <div class="space-y-3">
                        <div>
                            <label class="text-xs font-bold text-gray-500">تقرير (30) <span class="lang-en">Report</span></label>
                            <input type="number" id="s3_r" max="30" value="0" oninput="calc(3)" class="w-full p-2 border rounded-lg text-center font-black text-purple-700">
                        </div>
                        <div>
                            <label class="text-xs font-bold text-gray-500">عملي (40) <span class="lang-en">Practical</span></label>
                            <input type="number" id="s3_p" max="40" value="0" oninput="calc(3)" class="w-full p-2 border rounded-lg text-center font-black text-purple-700">
                        </div>
                        <div>
                            <label class="text-xs font-bold text-gray-500">مناقشة (30) <span class="lang-en">Presentation</span></label>
                            <input type="number" id="s3_d" max="30" value="0" oninput="calc(3)" class="w-full p-2 border rounded-lg text-center font-black text-purple-700">
                        </div>
                    </div>
                    <div class="pt-4 border-t border-purple-200 text-center">
                        <span class="text-xs text-gray-400 block">المجموع / Total</span>
                        <span id="total3" class="text-3xl font-black text-purple-800">0</span>
                    </div>
                </div>
            </div>

            <!-- Global Decision -->
            <div class="bg-gray-900 p-6 rounded-2xl text-white flex flex-col md:flex-row gap-6 items-center justify-between">
                <div>
                    <p class="font-bold text-yellow-500">ملاحظة عامة:</p>
                    <p class="text-sm opacity-70">يتم تقييم كل طالب بناءً على مجهوده الفردي ومشاركته في المشروع.</p>
                </div>
                <div class="w-full md:w-auto">
                    <button type="button" onclick="window.print()" class="w-full bg-indigo-600 px-8 py-4 rounded-xl font-bold hover:bg-indigo-500 transition-colors no-print">
                        طباعة النتائج / Print PDF
                    </button>
                </div>
            </div>

            <!-- Actions Grid -->
            <div class="grid grid-cols-1 md:grid-cols-3 gap-4 no-print">
                <button type="button" onclick="exportExcel()" class="bg-emerald-600 text-white p-4 rounded-xl font-bold hover:bg-emerald-700">Excel Export</button>
                <button type="button" id="saveCloudBtn" onclick="saveToCloud()" class="bg-blue-600 text-white p-4 rounded-xl font-bold hover:bg-blue-700">Cloud Save</button>
                <button type="button" onclick="sendWhatsApp()" class="bg-green-600 text-white p-4 rounded-xl font-bold hover:bg-green-700">WhatsApp Send</button>
            </div>
        </form>
    </div>

    <script>
        function calc(id) {
            const r = parseInt(document.getElementById(`s${id}_r`).value) || 0;
            const p = parseInt(document.getElementById(`s${id}_p`).value) || 0;
            const d = parseInt(document.getElementById(`s${id}_d`).value) || 0;
            
            // Limit checks
            if (r > 30) document.getElementById(`s${id}_r`).value = 30;
            if (p > 40) document.getElementById(`s${id}_p`).value = 40;
            if (d > 30) document.getElementById(`s${id}_d`).value = 30;

            const total = Math.min(100, (parseInt(document.getElementById(`s${id}_r`).value) || 0) + 
                                       (parseInt(document.getElementById(`s${id}_p`).value) || 0) + 
                                       (parseInt(document.getElementById(`s${id}_d`).value) || 0));
            
            document.getElementById(`total${id}`).innerText = total;
        }

        function getFormData() {
            return {
                projectTitle: document.getElementById('projectTitle').value,
                supervisor: document.getElementById('supervisor').value,
                examiner: document.getElementById('examiner').value,
                date: document.getElementById('evalDate').value,
                students: [
                    { name: document.getElementById('name1').value, total: document.getElementById('total1').innerText },
                    { name: document.getElementById('name2').value, total: document.getElementById('total2').innerText },
                    { name: document.getElementById('name3').value, total: document.getElementById('total3').innerText }
                ]
            };
        }

        function exportExcel() {
            const data = getFormData();
            const wsData = [
                ["Graduation Project Evaluation", ""],
                ["Project Title", data.projectTitle],
                ["Supervisor", data.supervisor],
                ["Examiner", data.examiner],
                ["Date", data.date],
                ["", ""],
                ["Student Name", "Total Grade (100)"]
            ];
            data.students.forEach(s => {
                if(s.name) wsData.push([s.name, s.total]);
            });

            const wb = XLSX.utils.book_new();
            const ws = XLSX.utils.aoa_to_sheet(wsData);
            XLSX.utils.book_append_sheet(wb, ws, "Scores");
            XLSX.writeFile(wb, "Project_Grades.xlsx");
        }

        function sendWhatsApp() {
            const data = getFormData();
            let msg = `*Graduation Project Scores*%0A` +
                      `Project: ${data.projectTitle}%0A` +
                      `-----------------------%0A`;
            data.students.forEach(s => {
                if(s.name) msg += `${s.name}: *${s.total}/100*%0A`;
            });
            window.open(`https://wa.me/970592263191?text=${msg}`, '_blank');
        }
    </script>
</body>
</html>
