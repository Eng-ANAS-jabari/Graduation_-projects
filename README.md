<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>نظام تقييم مشاريع التخرج الرقمي | Graduation Project Evaluation System</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;500;700&display=swap" rel="stylesheet">
    <!-- SheetJS for Excel Export -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <!-- Firebase Libraries -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, serverTimestamp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const firebaseConfig = JSON.parse(__firebase_config);
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'grad-eval-app';

        // Auth Logic
        async function initAuth() {
            try {
                await signInAnonymously(auth);
            } catch (error) {
                console.error("Auth failed", error);
            }
        }
        initAuth();

        window.saveToCloud = async function() {
            const user = auth.currentUser;
            if (!user) {
                alert("يرجى الانتظار حتى يتم الاتصال بالنظام... / Please wait for system connection...");
                return;
            }

            const data = getFormData();
            if (!data.studentName) {
                alert("يرجى إدخال اسم الطالب على الأقل / Please enter at least the student name");
                return;
            }

            try {
                const btn = document.getElementById('saveCloudBtn');
                btn.disabled = true;
                btn.innerText = "جاري الحفظ... / Saving...";
                
                await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'evaluations'), {
                    ...data,
                    timestamp: serverTimestamp(),
                    userId: user.uid
                });

                alert("تم حفظ البيانات في السحابة بنجاح! / Data saved to cloud successfully!");
            } catch (e) {
                alert("حدث خطأ أثناء الحفظ. / An error occurred while saving.");
            } finally {
                const btn = document.getElementById('saveCloudBtn');
                btn.disabled = false;
                btn.innerText = "حفظ سحابي / Cloud Save";
            }
        };
    </script>
    <style>
        body { font-family: 'Tajawal', sans-serif; background-color: #f8fafc; }
        .lang-en { display: block; font-size: 0.85rem; opacity: 0.7; font-weight: normal; }
        @media print { .no-print { display: none; } body { background: white; } .container { box-shadow: none; border: none; } }
    </style>
</head>
<body class="p-4 md:p-8">

    <div class="max-w-5xl mx-auto bg-white shadow-xl rounded-3xl overflow-hidden container border border-gray-100">
        <!-- Header -->
        <div class="bg-gradient-to-r from-indigo-900 to-blue-800 p-10 text-white text-center">
            <h1 class="text-3xl font-bold mb-2">منصة تقييم مشاريع التخرج</h1>
            <p class="text-xl font-medium opacity-90">Graduation Project Evaluation Platform</p>
            <p class="text-blue-100 opacity-80 mt-2">نظام ذكي لتسجيل العلامات | Smart Grading System</p>
        </div>

        <form id="evaluationForm" class="p-8 space-y-10">
            <!-- Section 1: Basic Info -->
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 pb-8 border-b border-gray-100">
                <div class="space-y-2">
                    <label class="block font-bold text-gray-700">
                        اسم الطالب / المجموعة
                        <span class="lang-en">Student / Group Name</span>
                    </label>
                    <input type="text" id="studentName" class="w-full px-4 py-3 bg-gray-50 border rounded-xl focus:ring-2 focus:ring-indigo-500 outline-none transition-all" placeholder="الاسم الكامل / Full Name">
                </div>
                <div class="space-y-2">
                    <label class="block font-bold text-gray-700">
                        عنوان المشروع
                        <span class="lang-en">Project Title</span>
                    </label>
                    <input type="text" id="projectTitle" class="w-full px-4 py-3 bg-gray-50 border rounded-xl focus:ring-2 focus:ring-indigo-500 outline-none transition-all" placeholder="عنوان البحث / Research Title">
                </div>
                <div class="space-y-2">
                    <label class="block font-bold text-gray-700">
                        المشرف
                        <span class="lang-en">Supervisor</span>
                    </label>
                    <input type="text" id="supervisor" class="w-full px-4 py-3 bg-gray-50 border rounded-xl focus:ring-2 focus:ring-indigo-500 outline-none transition-all" placeholder="اسم المشرف / Supervisor Name">
                </div>
                <div class="space-y-2">
                    <label class="block font-bold text-gray-700">
                        اسم المناقش
                        <span class="lang-en">Examiner Name</span>
                    </label>
                    <input type="text" id="examiner" class="w-full px-4 py-3 bg-gray-50 border rounded-xl focus:ring-2 focus:ring-indigo-500 outline-none transition-all" placeholder="اسم المناقش / Examiner Name">
                </div>
                <div class="space-y-2">
                    <label class="block font-bold text-gray-700">
                        التاريخ
                        <span class="lang-en">Date</span>
                    </label>
                    <input type="date" id="evalDate" class="w-full px-4 py-3 bg-gray-50 border rounded-xl focus:ring-2 focus:ring-indigo-500 outline-none transition-all">
                </div>
            </div>

            <!-- Categories -->
            <div class="space-y-12">
                <!-- Group 1 -->
                <section>
                    <div class="flex items-center gap-3 mb-6 border-r-4 border-indigo-600 pr-4">
                        <div class="w-10 h-10 bg-indigo-600 text-white rounded-lg flex items-center justify-center font-bold">1</div>
                        <div>
                            <h3 class="text-xl font-bold text-gray-800">التقرير المكتوب (30 درجة)</h3>
                            <span class="lang-en">Written Report (30 Marks)</span>
                        </div>
                    </div>
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                        <div class="score-card bg-white p-5 border-2 border-gray-50 rounded-2xl shadow-sm hover:border-indigo-200 transition-colors">
                            <label class="block text-sm font-bold text-gray-500 mb-3">هيكل التقرير (10) <span class="lang-en">Report Structure</span></label>
                            <input type="number" id="s1_1" max="10" min="0" value="0" class="score-input w-full text-3xl font-black text-center text-indigo-700 outline-none" oninput="calculateTotal()">
                        </div>
                        <div class="score-card bg-white p-5 border-2 border-gray-50 rounded-2xl shadow-sm hover:border-indigo-200 transition-colors">
                            <label class="block text-sm font-bold text-gray-500 mb-3">المنهجية (10) <span class="lang-en">Methodology</span></label>
                            <input type="number" id="s1_2" max="10" min="0" value="0" class="score-input w-full text-3xl font-black text-center text-indigo-700 outline-none" oninput="calculateTotal()">
                        </div>
                        <div class="score-card bg-white p-5 border-2 border-gray-50 rounded-2xl shadow-sm hover:border-indigo-200 transition-colors">
                            <label class="block text-sm font-bold text-gray-500 mb-3">التوثيق (10) <span class="lang-en">Documentation</span></label>
                            <input type="number" id="s1_3" max="10" min="0" value="0" class="score-input w-full text-3xl font-black text-center text-indigo-700 outline-none" oninput="calculateTotal()">
                        </div>
                    </div>
                </section>

                <!-- Group 2 -->
                <section>
                    <div class="flex items-center gap-3 mb-6 border-r-4 border-blue-600 pr-4">
                        <div class="w-10 h-10 bg-blue-600 text-white rounded-lg flex items-center justify-center font-bold">2</div>
                        <div>
                            <h3 class="text-xl font-bold text-gray-800">الجانب العملي (40 درجة)</h3>
                            <span class="lang-en">Practical Part (40 Marks)</span>
                        </div>
                    </div>
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                        <div class="score-card bg-white p-5 border-2 border-gray-50 rounded-2xl shadow-sm hover:border-blue-200 transition-colors">
                            <label class="block text-sm font-bold text-gray-500 mb-3">الإبداع (10) <span class="lang-en">Creativity</span></label>
                            <input type="number" id="s2_1" max="10" min="0" value="0" class="score-input w-full text-3xl font-black text-center text-blue-700 outline-none" oninput="calculateTotal()">
                        </div>
                        <div class="score-card bg-white p-5 border-2 border-gray-50 rounded-2xl shadow-sm hover:border-blue-200 transition-colors">
                            <label class="block text-sm font-bold text-gray-500 mb-3">اكتمال التنفيذ (20) <span class="lang-en">Implementation</span></label>
                            <input type="number" id="s2_2" max="20" min="0" value="0" class="score-input w-full text-3xl font-black text-center text-blue-700 outline-none" oninput="calculateTotal()">
                        </div>
                        <div class="score-card bg-white p-5 border-2 border-gray-50 rounded-2xl shadow-sm hover:border-blue-200 transition-colors">
                            <label class="block text-sm font-bold text-gray-500 mb-3">الإخراج النهائي (10) <span class="lang-en">Final Output</span></label>
                            <input type="number" id="s2_3" max="10" min="0" value="0" class="score-input w-full text-3xl font-black text-center text-blue-700 outline-none" oninput="calculateTotal()">
                        </div>
                    </div>
                </section>

                <!-- Group 3 -->
                <section>
                    <div class="flex items-center gap-3 mb-6 border-r-4 border-emerald-600 pr-4">
                        <div class="w-10 h-10 bg-emerald-600 text-white rounded-lg flex items-center justify-center font-bold">3</div>
                        <div>
                            <h3 class="text-xl font-bold text-gray-800">المناقشة (30 درجة)</h3>
                            <span class="lang-en">Presentation & Oral (30 Marks)</span>
                        </div>
                    </div>
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                        <div class="score-card bg-white p-5 border-2 border-gray-50 rounded-2xl shadow-sm hover:border-emerald-200 transition-colors">
                            <label class="block text-sm font-bold text-gray-500 mb-3">مهارات العرض (10) <span class="lang-en">Presentation Skills</span></label>
                            <input type="number" id="s3_1" max="10" min="0" value="0" class="score-input w-full text-3xl font-black text-center text-emerald-700 outline-none" oninput="calculateTotal()">
                        </div>
                        <div class="score-card bg-white p-5 border-2 border-gray-50 rounded-2xl shadow-sm hover:border-emerald-200 transition-colors">
                            <label class="block text-sm font-bold text-gray-500 mb-3">التمكن العلمي (10) <span class="lang-en">Scientific Mastery</span></label>
                            <input type="number" id="s3_2" max="10" min="0" value="0" class="score-input w-full text-3xl font-black text-center text-emerald-700 outline-none" oninput="calculateTotal()">
                        </div>
                        <div class="score-card bg-white p-5 border-2 border-gray-50 rounded-2xl shadow-sm hover:border-emerald-200 transition-colors">
                            <label class="block text-sm font-bold text-gray-500 mb-3">ردود الفعل (10) <span class="lang-en">Q&A Handling</span></label>
                            <input type="number" id="s3_3" max="10" min="0" value="0" class="score-input w-full text-3xl font-black text-center text-emerald-700 outline-none" oninput="calculateTotal()">
                        </div>
                    </div>
                </section>
            </div>

            <!-- Result Box -->
            <div class="bg-slate-900 rounded-3xl p-10 text-white flex flex-col md:flex-row items-center justify-between gap-8">
                <div>
                    <h4 class="text-slate-400 font-bold mb-2">النتيجة النهائية | Final Result</h4>
                    <div class="text-7xl font-black" id="totalDisplay">0 <span class="text-2xl text-slate-500">/ 100</span></div>
                </div>
                <div class="w-full md:w-1/3 space-y-3">
                    <label class="font-bold text-sm text-slate-400 text-center block">القرار النهائي | Final Decision</label>
                    <select id="finalDecision" class="w-full p-4 bg-slate-800 border-none rounded-xl text-white outline-none focus:ring-2 focus:ring-indigo-500 text-center">
                        <option>ناجح بامتياز / Pass with Excellence</option>
                        <option>ناجح / Pass</option>
                        <option>ناجح مع تعديلات / Pass with Changes</option>
                        <option>راسب - إعادة / Fail - Retake</option>
                    </select>
                </div>
            </div>

            <!-- Actions -->
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 no-print">
                <button type="button" onclick="exportToExcel()" class="bg-emerald-600 hover:bg-emerald-700 text-white p-4 rounded-2xl font-bold shadow-lg transition-all flex flex-col items-center justify-center">
                    <span>تصدير Excel</span>
                    <span class="text-xs font-normal opacity-80">Export to Excel</span>
                </button>
                <button type="button" onclick="sendToWhatsApp()" class="bg-green-500 hover:bg-green-600 text-white p-4 rounded-2xl font-bold shadow-lg transition-all flex flex-col items-center justify-center">
                    <span>إرسال واتساب</span>
                    <span class="text-xs font-normal opacity-80">Send to WhatsApp</span>
                </button>
                <button type="button" id="saveCloudBtn" onclick="saveToCloud()" class="bg-indigo-600 hover:bg-indigo-700 text-white p-4 rounded-2xl font-bold shadow-lg transition-all flex flex-col items-center justify-center">
                    <span>حفظ سحابي</span>
                    <span class="text-xs font-normal opacity-80">Cloud Save</span>
                </button>
                <button type="button" onclick="window.print()" class="bg-slate-700 hover:bg-slate-800 text-white p-4 rounded-2xl font-bold shadow-lg transition-all flex flex-col items-center justify-center">
                    <span>طباعة PDF</span>
                    <span class="text-xs font-normal opacity-80">Print PDF</span>
                </button>
            </div>
        </form>
    </div>

    <script>
        function getFormData() {
            return {
                studentName: document.getElementById('studentName').value,
                projectTitle: document.getElementById('projectTitle').value,
                supervisor: document.getElementById('supervisor').value,
                examiner: document.getElementById('examiner').value,
                date: document.getElementById('evalDate').value,
                reportScore: (parseInt(document.getElementById('s1_1').value) || 0) + (parseInt(document.getElementById('s1_2').value) || 0) + (parseInt(document.getElementById('s1_3').value) || 0),
                practicalScore: (parseInt(document.getElementById('s2_1').value) || 0) + (parseInt(document.getElementById('s2_2').value) || 0) + (parseInt(document.getElementById('s2_3').value) || 0),
                presentationScore: (parseInt(document.getElementById('s3_1').value) || 0) + (parseInt(document.getElementById('s3_2').value) || 0) + (parseInt(document.getElementById('s3_3').value) || 0),
                total: parseInt(document.getElementById('totalDisplay').innerText),
                decision: document.getElementById('finalDecision').value
            };
        }

        function calculateTotal() {
            const inputs = document.querySelectorAll('.score-input');
            let total = 0;
            inputs.forEach(input => {
                const max = parseInt(input.getAttribute('max'));
                let val = parseInt(input.value) || 0;
                if (val > max) { val = max; input.value = max; }
                if (val < 0) { val = 0; input.value = 0; }
                total += val;
            });
            document.getElementById('totalDisplay').innerHTML = total + ' <span class="text-2xl text-slate-500">/ 100</span>';
        }

        function exportToExcel() {
            const data = getFormData();
            if(!data.studentName) { alert("أدخل اسم الطالب أولاً / Enter student name first"); return; }
            
            const wsData = [
                ["معيار التقييم / Evaluation Criteria", "البيانات أو الدرجة / Data or Mark"],
                ["اسم الطالب / Student Name", data.studentName],
                ["المشروع / Project", data.projectTitle],
                ["المشرف / Supervisor", data.supervisor],
                ["المناقش / Examiner", data.examiner],
                ["التاريخ / Date", data.date],
                ["", ""],
                ["درجة التقرير / Report Score", data.reportScore + " / 30"],
                ["درجة العملي / Practical Score", data.practicalScore + " / 40"],
                ["درجة المناقشة / Presentation Score", data.presentationScore + " / 30"],
                ["المجموع الكلي / Total", data.total + " / 100"],
                ["القرار النهائي / Final Decision", data.decision]
            ];

            const wb = XLSX.utils.book_new();
            const ws = XLSX.utils.aoa_to_sheet(wsData);
            XLSX.utils.book_append_sheet(wb, ws, "Evaluation");
            XLSX.writeFile(wb, `Evaluation_${data.studentName}.xlsx`);
        }

        function sendToWhatsApp() {
            const data = getFormData();
            if(!data.studentName) {
                alert("يرجى إدخال اسم الطالب / Please enter student name");
                return;
            }

            const phone = "970592263191";
            const message = `*Evaluation Form | نموذج تقييم*%0A` +
                            `--------------------------%0A` +
                            `*Student:* ${data.studentName}%0A` +
                            `*Project:* ${data.projectTitle}%0A` +
                            `*Supervisor:* ${data.supervisor}%0A` +
                            `*Examiner:* ${data.examiner}%0A` +
                            `--------------------------%0A` +
                            `*Report Score:* ${data.reportScore} / 30%0A` +
                            `*Practical Score:* ${data.practicalScore} / 40%0A` +
                            `*Presentation:* ${data.presentationScore} / 30%0A` +
                            `--------------------------%0A` +
                            `*Total Score:* ${data.total} / 100%0A` +
                            `*Decision:* ${data.decision}`;

            const wpUrl = `https://wa.me/${phone}?text=${message}`;
            window.open(wpUrl, '_blank');
        }
    </script>
</body>
</html>
