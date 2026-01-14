<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>نظام تقييم مشاريع التخرج | النسخة النهائية</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;500;700&display=swap" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        body { 
            font-family: 'Tajawal', sans-serif; 
            background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
            min-height: 100vh;
        }
        
        .score-input { 
            border: 2px solid #e2e8f0; 
            transition: all 0.2s; 
            text-align: center; 
            font-weight: 700; 
            font-size: 1.1rem; 
        }
        
        .score-input:focus { 
            border-color: #4f46e5; 
            outline: none; 
            background-color: #fffbeb; 
            box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.1);
        }
        
        .student-card {
            transition: all 0.3s ease;
            border: 2px solid transparent;
        }
        
        .student-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
            border-color: #4f46e5;
        }
        
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }
        
        .fade-in {
            animation: fadeIn 0.5s ease forwards;
        }
        
        .whatsapp-modal {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.7);
            display: none;
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }
        
        .whatsapp-modal-content {
            background: white;
            padding: 30px;
            border-radius: 20px;
            width: 90%;
            max-width: 500px;
            max-height: 80vh;
            overflow-y: auto;
        }
        
        .admin-student-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 10px;
            margin: 5px 0;
            background: #f8fafc;
            border-radius: 10px;
            border: 1px solid #e2e8f0;
        }
        
        @media print { 
            .no-print { display: none !important; } 
            body { padding: 0 !important; background: white !important; } 
            .container { box-shadow: none !important; border: none !important; }
            .student-card { border: 1px solid #eee !important; break-inside: avoid; }
        }
    </style>
</head>
<body class="p-4 md:p-8">

    <div id="app" class="max-w-6xl mx-auto space-y-6">
        
        <!-- واجهة اختيار الدور -->
        <div id="roleSelection" class="bg-white p-10 rounded-[2.5rem] shadow-2xl text-center no-print border border-slate-200 fade-in">
            <h2 class="text-3xl font-black mb-2 text-slate-800">نظام تقييم مشاريع التخرج</h2>
            <p class="text-slate-500 mb-10">إدارة التقييمات، توزيع الدرجات، واستخراج التقارير النهائية</p>
            
            <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                <button onclick="requestAdminAccess()" class="group p-8 bg-slate-50 border-4 border-slate-200 rounded-[2.5rem] hover:bg-slate-900 hover:text-white transition-all duration-300 shadow-lg">
                    <div class="text-4xl mb-4">🔐</div>
                    <div class="text-xl font-black">الإدارة والبيانات</div>
                    <p class="text-sm mt-2 text-slate-500 group-hover:text-slate-300">كلمة المرور: admin</p>
                </button>

                <button onclick="setRole('supervisor')" class="group p-8 bg-white border-4 border-indigo-600 rounded-[2.5rem] hover:bg-indigo-600 hover:text-white transition-all duration-300 shadow-xl">
                    <div class="text-4xl mb-4">📝</div>
                    <div class="text-xl font-black">تقييم المشرف</div>
                    <p class="text-sm mt-2 text-slate-500 group-hover:text-slate-300">التقييم التحضيري والتنفيذي</p>
                </button>
                
                <button onclick="setRole('examiner')" class="group p-8 bg-white border-4 border-emerald-600 rounded-[2.5rem] hover:bg-emerald-600 hover:text-white transition-all duration-300 shadow-xl">
                    <div class="text-4xl mb-4">🎓</div>
                    <div class="text-xl font-black">تقييم المناقش</div>
                    <p class="text-sm mt-2 text-slate-500 group-hover:text-slate-300">التقييم النهائي والمناقشة</p>
                </button>
            </div>
            
            <!-- إحصائيات النظام -->
            <div class="mt-12 pt-8 border-t border-slate-200">
                <h3 class="text-lg font-bold text-slate-700 mb-4">📊 إحصائيات النظام</h3>
                <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
                    <div class="bg-indigo-50 p-4 rounded-2xl">
                        <div class="text-xl font-bold text-indigo-700" id="totalProjects">0</div>
                        <div class="text-sm text-indigo-600">مشروع</div>
                    </div>
                    <div class="bg-emerald-50 p-4 rounded-2xl">
                        <div class="text-xl font-bold text-emerald-700" id="totalStudents">0</div>
                        <div class="text-sm text-emerald-600">طالب</div>
                    </div>
                    <div class="bg-amber-50 p-4 rounded-2xl">
                        <div class="text-xl font-bold text-amber-700" id="totalEvaluations">0</div>
                        <div class="text-sm text-amber-600">تقييم</div>
                    </div>
                    <div class="bg-purple-50 p-4 rounded-2xl">
                        <div class="text-xl font-bold text-purple-700" id="sharedUsers">0</div>
                        <div class="text-sm text-purple-600">مستخدم</div>
                    </div>
                </div>
            </div>
        </div>

        <!-- لوحة التحكم (المسؤول) -->
        <div id="adminPanel" class="hidden bg-white shadow-2xl rounded-[2.5rem] overflow-hidden border border-slate-200 fade-in">
            <div class="bg-gradient-to-r from-slate-900 to-slate-800 p-6 text-white flex justify-between items-center">
                <div>
                    <h2 class="text-2xl font-bold">لوحة التحكم المركزية</h2>
                    <p class="text-sm opacity-80 mt-1">إدارة المشاريع والطلاب وتوزيعها</p>
                </div>
                <button onclick="goBack()" class="bg-white/20 hover:bg-white/30 px-4 py-2 rounded-lg text-sm transition-all">
                    <i class="fas fa-arrow-left ml-2"></i> رجوع
                </button>
            </div>
            <div class="p-8 space-y-8">
                <!-- أدوات الاستيراد -->
                <div class="bg-gradient-to-r from-indigo-50 to-white p-8 rounded-3xl border-2 border-dashed border-indigo-200">
                    <div class="flex items-center gap-4 mb-6">
                        <div class="w-12 h-12 bg-indigo-100 rounded-2xl flex items-center justify-center">
                            <i class="fas fa-file-excel text-xl text-indigo-600"></i>
                        </div>
                        <div>
                            <h3 class="font-bold text-indigo-800 text-lg">📁 استيراد بيانات المشاريع</h3>
                            <p class="text-sm text-indigo-600">ارفع ملف Excel لتوزيع الطلاب على المشاريع تلقائياً</p>
                        </div>
                    </div>
                    <input type="file" id="excelUpload" accept=".xlsx, .xls" class="hidden" onchange="importExcel(event)">
                    <button onclick="document.getElementById('excelUpload').click()" class="bg-indigo-600 text-white px-8 py-3 rounded-2xl font-bold shadow-lg hover:bg-indigo-700 transition-all">
                        <i class="fas fa-upload ml-2"></i> رفع ملف Excel
                    </button>
                </div>
                
                <!-- إدارة الطلاب المشتركين -->
                <div class="bg-gradient-to-r from-emerald-50 to-white p-8 rounded-3xl border border-emerald-100">
                    <h3 class="font-bold text-emerald-800 text-lg mb-6 flex items-center gap-3">
                        <i class="fas fa-users"></i> إدارة أسماء الطلاب المشتركين
                    </h3>
                    <div class="space-y-4">
                        <div class="flex gap-3">
                            <input type="text" id="newStudentName" placeholder="أدخل اسم طالب جديد" class="flex-1 p-3 bg-white border border-slate-200 rounded-xl outline-none focus:border-emerald-500">
                            <button onclick="addSharedStudent()" class="bg-emerald-600 text-white px-6 py-3 rounded-xl font-bold hover:bg-emerald-700 transition-all">
                                <i class="fas fa-plus ml-2"></i> إضافة
                            </button>
                        </div>
                        <div class="mt-4">
                            <div class="text-sm font-bold text-slate-600 mb-3">قائمة الطلاب المحفوظة:</div>
                            <div id="sharedStudentsList" class="space-y-2 max-h-40 overflow-y-auto p-3 bg-slate-50 rounded-xl">
                                <!-- سيتم تعبئتها تلقائياً -->
                            </div>
                        </div>
                    </div>
                </div>

                <!-- قائمة المشاريع -->
                <div>
                    <h3 class="font-bold text-slate-800 text-xl mb-6 flex items-center gap-3">
                        <i class="fas fa-list-ul"></i> قائمة المشاريع المسجلة
                        <span class="text-sm font-normal text-slate-500 bg-slate-100 px-3 py-1 rounded-full" id="projectsCount">0 مشروع</span>
                    </h3>
                    <div id="adminDataList" class="grid grid-cols-1 md:grid-cols-2 gap-6">
                        <!-- سيتم تعبئتها ديناميكياً -->
                    </div>
                </div>
            </div>
        </div>

        <!-- نموذج التقييم الرئيسي -->
        <div id="mainContainer" class="hidden bg-white shadow-2xl rounded-[2.5rem] overflow-hidden border border-slate-200 fade-in">
            <!-- الهيدر -->
            <div id="formHeader" class="p-10 text-white text-center relative">
                <button onclick="goBack()" class="absolute top-8 left-8 bg-white/20 px-4 py-2 rounded-full text-xs font-bold hover:bg-white/30 transition-all no-print">
                    <i class="fas fa-arrow-left ml-2"></i> الرئيسية
                </button>
                <h1 id="headerTitle" class="text-4xl font-black"></h1>
                <p id="headerSubtitle" class="mt-2 opacity-80 font-medium"></p>
            </div>

            <form id="evaluationForm" class="p-8 md:p-12 space-y-12">
                <!-- اختيار المشروع والبيانات العامة -->
                <div class="grid grid-cols-1 md:grid-cols-3 gap-8 p-8 bg-slate-50 rounded-3xl border border-slate-100">
                    <div class="space-y-2">
                        <label class="block font-black text-slate-700 text-sm">اسم المشروع المختار</label>
                        <select id="projectSelect" class="w-full p-3 bg-white border border-slate-200 rounded-xl outline-none font-bold text-indigo-600 shadow-sm" onchange="handleProjectChange()">
                            <option value="">-- اختر من القائمة --</option>
                        </select>
                    </div>
                    <div id="dynamicFields" class="contents"></div>
                </div>

                <!-- شبكة تقييم الطلاب -->
                <div class="grid grid-cols-1 lg:grid-cols-2 xl:grid-cols-3 gap-8" id="studentsWrapper">
                    <div class="col-span-full py-20 text-center opacity-40">
                        <div class="text-5xl mb-4">🔍</div>
                        <p class="font-bold">يرجى اختيار المشروع لعرض الطلاب وبدء عملية التقييم</p>
                    </div>
                </div>

                <!-- إجراءات الحفظ والتصدير -->
                <div class="pt-10 flex flex-wrap justify-center gap-4 border-t border-slate-100 no-print">
                    <button type="button" onclick="exportToExcel()" class="bg-slate-100 hover:bg-slate-200 text-slate-700 px-8 py-4 rounded-2xl font-black transition-all shadow-sm">
                        <i class="fas fa-file-excel ml-2"></i> تصدير Excel
                    </button>
                    <button type="button" onclick="showWhatsAppModal()" class="bg-green-500 hover:bg-green-600 text-white px-8 py-4 rounded-2xl font-black transition-all shadow-lg">
                        <i class="fab fa-whatsapp ml-2"></i> إرسال عبر WhatsApp
                    </button>
                    <button type="button" onclick="window.print()" class="bg-indigo-600 hover:bg-indigo-700 text-white px-10 py-4 rounded-2xl font-black transition-all shadow-xl">
                        <i class="fas fa-print ml-2"></i> طباعة التقرير
                    </button>
                </div>
            </form>
        </div>
    </div>

    <!-- قالب بطاقة الطالب -->
    <template id="studentTemplate">
        <div class="student-card bg-white border border-slate-200 rounded-[2.5rem] p-8 shadow-sm hover:shadow-md transition-all flex flex-col h-full fade-in">
            <div class="flex justify-between items-start mb-6">
                <div>
                    <span class="text-[10px] font-black text-slate-400 uppercase tracking-widest">طالب مشروع تخرج</span>
                    <div class="flex items-center gap-3 mt-1">
                        <h4 class="student-name-display text-2xl font-black text-slate-800"></h4>
                        <button type="button" onclick="changeStudentName(this)" class="text-slate-400 hover:text-indigo-600 text-sm no-print">
                            <i class="fas fa-edit"></i>
                        </button>
                    </div>
                </div>
                <div class="text-3xl">👤</div>
            </div>
            <div class="criteria-list space-y-5 flex-grow"></div>
            <div class="mt-10 pt-6 border-t border-slate-100 flex justify-between items-end">
                <div>
                    <span class="text-[10px] font-black text-slate-400 block mb-1">الدرجة النهائية</span>
                    <div class="flex items-baseline gap-1">
                        <span class="text-4xl font-black text-indigo-600 student-total-display">0</span>
                        <span class="text-sm font-bold text-slate-400">/ 100</span>
                    </div>
                </div>
                <div class="student-result-text font-black text-xs px-5 py-2 rounded-full bg-slate-100 text-slate-500 uppercase tracking-wide">قيد التقييم</div>
            </div>
        </div>
    </template>

    <!-- نافذة اختيار اسم الطالب -->
    <div id="studentNameModal" class="fixed inset-0 bg-black/50 z-50 items-center justify-center hidden">
        <div class="bg-white rounded-3xl p-8 max-w-md w-full mx-4">
            <h3 class="text-xl font-bold text-slate-800 mb-6">اختيار اسم الطالب</h3>
            <div class="space-y-4">
                <div class="relative">
                    <input type="text" id="studentSearch" placeholder="ابحث عن اسم طالب..." class="w-full p-3 bg-slate-50 border border-slate-200 rounded-xl outline-none">
                    <i class="fas fa-search absolute left-3 top-3 text-slate-400"></i>
                </div>
                <div id="suggestedStudents" class="max-h-60 overflow-y-auto space-y-2">
                    <!-- سيتم تعبئتها تلقائياً -->
                </div>
                <div class="pt-4 border-t border-slate-200">
                    <input type="text" id="customStudentName" placeholder="أو أدخل اسم مخصص..." class="w-full p-3 bg-slate-50 border border-slate-200 rounded-xl outline-none mb-4">
                    <div class="flex justify-end gap-3">
                        <button onclick="closeStudentModal()" class="px-4 py-2 border border-slate-300 rounded-lg hover:bg-slate-50">إلغاء</button>
                        <button onclick="saveStudentName()" class="bg-indigo-600 text-white px-6 py-2 rounded-lg font-bold hover:bg-indigo-700">حفظ</button>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- نافذة واتساب -->
    <div id="whatsappModal" class="whatsapp-modal">
        <div class="whatsapp-modal-content">
            <div class="flex justify-between items-center mb-6">
                <h3 class="text-xl font-bold text-slate-800">إرسال النتائج عبر واتساب</h3>
                <button onclick="closeWhatsAppModal()" class="text-slate-500 hover:text-slate-700">
                    <i class="fas fa-times text-xl"></i>
                </button>
            </div>
            
            <div class="space-y-4 mb-6">
                <div class="bg-slate-50 p-4 rounded-xl">
                    <label class="block text-sm font-bold text-slate-700 mb-2">👨‍🏫 اسم المقيم</label>
                    <input type="text" id="evaluatorName" class="w-full p-3 bg-white border border-slate-200 rounded-lg outline-none" placeholder="أدخل اسمك">
                </div>
                
                <div class="bg-slate-50 p-4 rounded-xl">
                    <label class="block text-sm font-bold text-slate-700 mb-2">📱 رقم الهاتف (اختياري)</label>
                    <input type="text" id="phoneNumber" class="w-full p-3 bg-white border border-slate-200 rounded-lg outline-none" placeholder="مثال: 966500000000">
                </div>
                
                <div class="bg-slate-50 p-4 rounded-xl">
                    <label class="block text-sm font-bold text-slate-700 mb-2">💬 تخصيص الرسالة</label>
                    <textarea id="customMessage" class="w-full p-3 bg-white border border-slate-200 rounded-lg outline-none h-32" placeholder="أضف ملاحظات إضافية هنا..."></textarea>
                </div>
            </div>
            
            <div class="mb-6">
                <h4 class="font-bold text-slate-700 mb-3">معاينة الرسالة:</h4>
                <div id="messagePreview" class="bg-green-50 border border-green-200 rounded-xl p-4 text-sm text-slate-700 whitespace-pre-line max-h-40 overflow-y-auto">
                    سيظهر محتوى الرسالة هنا...
                </div>
            </div>
            
            <div class="flex gap-3">
                <button onclick="closeWhatsAppModal()" class="flex-1 bg-slate-100 hover:bg-slate-200 text-slate-700 px-6 py-3 rounded-xl font-bold transition-all">
                    إلغاء
                </button>
                <button onclick="sendWhatsAppMessage()" class="flex-1 bg-green-500 hover:bg-green-600 text-white px-6 py-3 rounded-xl font-bold transition-all flex items-center justify-center gap-2">
                    <i class="fab fa-whatsapp"></i>
                    إرسال الآن
                </button>
            </div>
        </div>
    </div>

    <script>
        // قاعدة البيانات المحلية مع دعم المشاركين
        let db = JSON.parse(localStorage.getItem('grad_db_shared')) || {
            projects: [],
            sharedStudents: ["أحمد محمد", "سارة محمود", "خالد عبد الله", "فاطمة علي", "محمد حسن"],
            evaluations: {},
            lastModified: new Date().toISOString()
        };
        
        let currentRole = '';
        let currentProject = null;
        let currentStudentElement = null;

        const roles = {
            supervisor: { 
                title: "تقييم المشرف الأكاديمي", 
                subtitle: "التقييم التحضيري والتنفيذي", 
                color: "bg-gradient-to-r from-indigo-600 to-indigo-800",
                roleName: "المشرف",
                criteria: [
                    {id:'book',label:'توثيق البحث',max:25},
                    {id:'practical',label:'التنفيذ العملي',max:35},
                    {id:'meetings',label:'الحضور والمتابعة',max:20},
                    {id:'ethics',label:'أخلاقيات العمل',max:20}
                ] 
            },
            examiner: { 
                title: "تقييم لجنة المناقشة", 
                subtitle: "التقييم النهائي والمناقشة العلمية", 
                color: "bg-gradient-to-r from-emerald-600 to-emerald-800",
                roleName: "المناقش",
                criteria: [
                    {id:'report',label:'جودة التقرير',max:25},
                    {id:'logic',label:'المنطق البرمجي',max:25},
                    {id:'defense',label:'قوة المناقشة',max:25},
                    {id:'presentation',label:'العرض المرئي',max:25}
                ] 
            }
        };

        // تحميل الإحصائيات
        function updateStats() {
            document.getElementById('totalProjects').textContent = db.projects.length;
            document.getElementById('totalStudents').textContent = db.sharedStudents.length;
            
            // حساب التقييمات
            let evalCount = 0;
            for (let key in db.evaluations) {
                if (db.evaluations[key]) evalCount++;
            }
            document.getElementById('totalEvaluations').textContent = evalCount;
            
            // حساب المستخدمين الفريدين
            let users = JSON.parse(localStorage.getItem('unique_users')) || [];
            document.getElementById('sharedUsers').textContent = users.length || 1;
        }

        // وظائف اختيار الدور
        function setRole(role) {
            currentRole = role;
            const cfg = roles[role];
            
            document.getElementById('roleSelection').classList.add('hidden');
            document.getElementById('mainContainer').classList.remove('hidden');
            document.getElementById('formHeader').className = `p-10 text-white text-center relative ${cfg.color}`;
            document.getElementById('headerTitle').innerText = cfg.title;
            document.getElementById('headerSubtitle').innerText = cfg.subtitle;
            
            // تعبئة قائمة المشاريع
            const sel = document.getElementById('projectSelect');
            sel.innerHTML = '<option value="">-- اختر المشروع --</option>' + 
                db.projects.map(p => `<option value="${p.id}">${p.title}</option>`).join('');
            
            // إعداد الحقول الديناميكية
            const dyn = document.getElementById('dynamicFields');
            dyn.innerHTML = `
                <div class="space-y-2">
                    <label class="block font-black text-slate-700 text-sm">المشرف المسؤول</label>
                    <input type="text" id="supervisorName" class="w-full p-3 bg-white border border-slate-200 rounded-xl outline-none font-bold" readonly>
                </div>
                <div class="space-y-2">
                    <label class="block font-black text-slate-700 text-sm">التاريخ</label>
                    <input type="date" id="evalDate" class="w-full p-3 bg-white border border-slate-200 rounded-xl outline-none font-bold" value="${new Date().toISOString().split('T')[0]}">
                </div>`;
            
            // تسجيل استخدام النظام
            registerUser();
            showNotification(`مرحباً بك في ${cfg.title}`, 'success');
        }

        // التعامل مع تغيير المشروع
        function handleProjectChange() {
            const projectId = document.getElementById('projectSelect').value;
            const project = db.projects.find(p => p.id === projectId);
            const wrap = document.getElementById('studentsWrapper');
            
            if(!project) { 
                wrap.innerHTML = `
                    <div class="col-span-full py-20 text-center opacity-40">
                        <div class="text-5xl mb-4">🔍</div>
                        <p class="font-bold">يرجى اختيار المشروع</p>
                    </div>`; 
                return; 
            }
            
            currentProject = project;
            document.getElementById('supervisorName').value = project.supervisor;
            wrap.innerHTML = '';
            const criteria = roles[currentRole].criteria;

            // استخدام الطلاب المشتركين إذا لم يكن هناك طلاب محددين للمشروع
            const students = project.students && project.students.length > 0 ? project.students : db.sharedStudents;
            
            students.forEach((name, index) => {
                const temp = document.getElementById('studentTemplate').content.cloneNode(true);
                const card = temp.querySelector('.student-card');
                card.dataset.studentIndex = index;
                
                // تعيين اسم الطالب
                const nameDisplay = card.querySelector('.student-name-display');
                nameDisplay.textContent = name;
                
                // تحميل التقييمات السابقة إن وجدت
                const studentKey = `${projectId}_${index}`;
                const previousScores = db.evaluations[studentKey]?.[currentRole] || {};
                
                // إضافة معايير التقييم
                criteria.forEach(crit => {
                    const row = document.createElement('div');
                    row.innerHTML = `
                        <div class="flex justify-between text-[10px] font-black text-slate-400 mb-2 uppercase tracking-tighter">
                            <span>${crit.label}</span>
                            <span>الحد الأقصى: ${crit.max}</span>
                        </div>
                        <input type="number" min="0" max="${crit.max}" value="${previousScores[crit.id] || 0}" 
                               class="score-input w-full p-2 rounded-2xl border" 
                               oninput="updateCardScore(this, ${crit.max}, '${studentKey}', '${crit.id}')">`;
                    card.querySelector('.criteria-list').appendChild(row);
                });
                
                wrap.appendChild(temp);
                updateCardTotal(card, studentKey);
            });
            
            showNotification(`تم تحميل ${students.length} طالب`, 'info');
        }

        // تحديث درجات الطالب
        function updateCardScore(input, max, studentKey, criteriaId) {
            let val = parseInt(input.value) || 0;
            if(val > max) input.value = max;
            if(val < 0) input.value = 0;
            
            // حفظ في قاعدة البيانات
            if (!db.evaluations[studentKey]) {
                db.evaluations[studentKey] = {};
            }
            if (!db.evaluations[studentKey][currentRole]) {
                db.evaluations[studentKey][currentRole] = {};
            }
            db.evaluations[studentKey][currentRole][criteriaId] = val;
            saveDatabase();
            
            // تحديث البطاقة
            const card = input.closest('.student-card');
            updateCardTotal(card, studentKey);
        }

        function updateCardTotal(card, studentKey) {
            const inputs = card.querySelectorAll('.score-input');
            let total = 0;
            inputs.forEach(i => total += (parseInt(i.value) || 0));
            
            // تحديث المجموع
            card.querySelector('.student-total-display').innerText = total;
            
            // تحديث التقدير
            const badge = card.querySelector('.student-result-text');
            if(total >= 90) { 
                badge.innerText = "ممتاز"; 
                badge.className = "student-result-text font-black text-xs px-5 py-2 rounded-full bg-indigo-100 text-indigo-700 uppercase tracking-wide"; 
            }
            else if(total >= 80) { 
                badge.innerText = "جيد جداً"; 
                badge.className = "student-result-text font-black text-xs px-5 py-2 rounded-full bg-emerald-100 text-emerald-700 uppercase tracking-wide"; 
            }
            else if(total >= 70) { 
                badge.innerText = "جيد"; 
                badge.className = "student-result-text font-black text-xs px-5 py-2 rounded-full bg-blue-100 text-blue-700 uppercase tracking-wide"; 
            }
            else if(total >= 60) { 
                badge.innerText = "مقبول"; 
                badge.className = "student-result-text font-black text-xs px-5 py-2 rounded-full bg-amber-100 text-amber-700 uppercase tracking-wide"; 
            }
            else if(total >= 50) { 
                badge.innerText = "ناجح"; 
                badge.className = "student-result-text font-black text-xs px-5 py-2 rounded-full bg-green-100 text-green-700 uppercase tracking-wide"; 
            }
            else { 
                badge.innerText = "راسب"; 
                badge.className = "student-result-text font-black text-xs px-5 py-2 rounded-full bg-rose-100 text-rose-700 uppercase tracking-wide"; 
            }
        }

        // تغيير اسم الطالب
        function changeStudentName(button) {
            currentStudentElement = button.closest('.student-card');
            const currentName = currentStudentElement.querySelector('.student-name-display').textContent;
            
            // إظهار النافذة
            document.getElementById('studentNameModal').classList.remove('hidden');
            document.getElementById('customStudentName').value = currentName;
            
            // تعبئة قائمة الاقتراحات
            const suggestionsDiv = document.getElementById('suggestedStudents');
            suggestionsDiv.innerHTML = '';
            
            // إضافة الطلاب المشتركين
            db.sharedStudents.forEach(student => {
                const div = document.createElement('div');
                div.className = 'p-3 hover:bg-slate-100 rounded-lg cursor-pointer transition-all';
                div.innerHTML = `
                    <div class="flex justify-between items-center">
                        <span class="font-medium">${student}</span>
                        <i class="fas fa-plus text-slate-400"></i>
                    </div>
                `;
                div.onclick = () => {
                    document.getElementById('customStudentName').value = student;
                };
                suggestionsDiv.appendChild(div);
            });
            
            // البحث في القائمة
            document.getElementById('studentSearch').addEventListener('input', function(e) {
                const searchTerm = e.target.value.toLowerCase();
                const suggestions = suggestionsDiv.querySelectorAll('div');
                
                suggestions.forEach(suggestion => {
                    const text = suggestion.textContent.toLowerCase();
                    suggestion.style.display = text.includes(searchTerm) ? 'block' : 'none';
                });
            });
        }

        function closeStudentModal() {
            document.getElementById('studentNameModal').classList.add('hidden');
            currentStudentElement = null;
        }

        function saveStudentName() {
            if (!currentStudentElement) return;
            
            const newName = document.getElementById('customStudentName').value.trim();
            if (!newName) {
                alert('يرجى إدخال اسم صحيح');
                return;
            }
            
            // تحديث الاسم في الواجهة
            currentStudentElement.querySelector('.student-name-display').textContent = newName;
            
            // إضافة إلى قائمة الطلاب المشتركين إذا كان جديداً
            if (!db.sharedStudents.includes(newName)) {
                db.sharedStudents.push(newName);
                saveDatabase();
                updateStats();
                showNotification('تم إضافة الاسم إلى القائمة المشتركة', 'success');
            }
            
            closeStudentModal();
        }

        // الإدارة والبيانات
        function requestAdminAccess() {
            const password = prompt("أدخل كلمة مرور الإدارة:");
            if (password === "admin") {
                showSection('admin');
            } else {
                alert("عذراً، كلمة المرور خاطئة");
            }
        }

        function showSection(id) {
            document.getElementById('roleSelection').classList.add('hidden');
            document.getElementById('adminPanel').classList.toggle('hidden', id !== 'admin');
            document.getElementById('mainContainer').classList.toggle('hidden', id !== 'main');
            
            if (id === 'admin') {
                renderAdminData();
            }
        }

        function goBack() {
            document.getElementById('adminPanel').classList.add('hidden');
            document.getElementById('mainContainer').classList.add('hidden');
            document.getElementById('roleSelection').classList.remove('hidden');
            updateStats();
        }

        function renderAdminData() {
            const list = document.getElementById('adminDataList');
            const count = document.getElementById('projectsCount');
            
            count.textContent = `${db.projects.length} مشروع`;
            
            // تحديث قائمة الطلاب المشتركين
            const studentsList = document.getElementById('sharedStudentsList');
            studentsList.innerHTML = db.sharedStudents.map(student => `
                <div class="admin-student-item">
                    <span class="font-medium">${student}</span>
                    <button onclick="removeSharedStudent('${student}')" class="text-rose-500 hover:text-rose-700 text-sm">
                        <i class="fas fa-times"></i>
                    </button>
                </div>
            `).join('');
            
            // تحديث قائمة المشاريع
            list.innerHTML = db.projects.map(p => `
                <div class="bg-slate-50 border p-6 rounded-3xl shadow-sm">
                    <div class="flex justify-between items-start mb-4">
                        <div>
                            <h5 class="font-black text-indigo-700 mb-1 text-lg">${p.title}</h5>
                            <p class="text-sm text-slate-500">إشراف: ${p.supervisor}</p>
                            <div class="flex items-center gap-2 mt-2">
                                <span class="text-xs font-bold px-3 py-1 rounded-full bg-indigo-100 text-indigo-700">${p.year || 2024}</span>
                            </div>
                        </div>
                        <button onclick="deleteProject('${p.id}')" class="text-rose-500 hover:text-rose-700">
                            <i class="fas fa-trash"></i>
                        </button>
                    </div>
                    <div class="mt-4">
                        <div class="text-xs font-bold text-slate-400 mb-2">الطلاب المسجلين:</div>
                        <div class="flex flex-wrap gap-2">
                            ${(p.students && p.students.length > 0 ? p.students : db.sharedStudents.slice(0, 3)).map(s => `
                                <span class="bg-white border text-xs font-bold px-3 py-1 rounded-full text-slate-600">${s}</span>
                            `).join('')}
                            ${p.students && p.students.length > 3 ? `<span class="text-xs text-slate-500">+ ${p.students.length - 3} أكثر</span>` : ''}
                        </div>
                    </div>
                    <div class="mt-6 pt-4 border-t border-slate-100 flex justify-between items-center">
                        <span class="text-xs text-slate-500">${p.students ? p.students.length : db.sharedStudents.length} طالب</span>
                        <button onclick="editProject('${p.id}')" class="text-primary hover:text-indigo-700 text-sm font-bold">
                            <i class="fas fa-edit ml-1"></i> تعديل
                        </button>
                    </div>
                </div>
            `).join('');
        }

        function addSharedStudent() {
            const input = document.getElementById('newStudentName');
            const name = input.value.trim();
            
            if (!name) {
                alert('يرجى إدخال اسم الطالب');
                return;
            }
            
            if (!db.sharedStudents.includes(name)) {
                db.sharedStudents.push(name);
                saveDatabase();
                renderAdminData();
                updateStats();
                input.value = '';
                showNotification('تم إضافة الطالب إلى القائمة المشتركة', 'success');
            } else {
                alert('هذا الاسم موجود بالفعل في القائمة');
            }
        }

        function removeSharedStudent(studentName) {
            if (confirm(`هل تريد حذف "${studentName}" من القائمة؟`)) {
                db.sharedStudents = db.sharedStudents.filter(s => s !== studentName);
                saveDatabase();
                renderAdminData();
                updateStats();
                showNotification('تم حذف الطالب من القائمة', 'warning');
            }
        }

        function deleteProject(projectId) {
            if (confirm('هل أنت متأكد من حذف هذا المشروع؟')) {
                db.projects = db.projects.filter(p => p.id !== projectId);
                saveDatabase();
                renderAdminData();
                updateStats();
                showNotification('تم حذف المشروع', 'warning');
            }
        }

        function editProject(projectId) {
            const project = db.projects.find(p => p.id === projectId);
            if (!project) return;
            
            const newTitle = prompt('تعديل عنوان المشروع:', project.title);
            if (newTitle) {
                project.title = newTitle;
                saveDatabase();
                renderAdminData();
                showNotification('تم تعديل المشروع', 'info');
            }
        }

        function importExcel(e) {
            const file = e.target.files[0];
            const reader = new FileReader();
            reader.onload = (event) => {
                try {
                    const workbook = XLSX.read(new Uint8Array(event.target.result), { type: 'array' });
                    const sheet = workbook.Sheets[workbook.SheetNames[0]];
                    const json = XLSX.utils.sheet_to_json(sheet);
                    
                    json.forEach(r => {
                        const p = r['اسم المشروع'] || r['project'] || r['Project'];
                        const s = r['اسم الطالب'] || r['student'] || r['Student'];
                        const sup = r['اسم المشرف'] || r['supervisor'] || r['Supervisor'];
                        
                        if(!p || !s) return;
                        
                        let project = db.projects.find(item => item.title === p);
                        if(project) { 
                            if(!project.students) project.students = [];
                            if(!project.students.includes(s)) project.students.push(s); 
                        }
                        else db.projects.push({ 
                            id: Date.now().toString() + Math.random(),
                            title: p, 
                            supervisor: sup || "غير معروف", 
                            students: [s],
                            year: new Date().getFullYear()
                        });
                    });
                    
                    saveDatabase();
                    renderAdminData();
                    updateStats();
                    showNotification(`تم استيراد ${json.length} سجل بنجاح!`, 'success');
                    
                } catch (error) {
                    console.error('Import error:', error);
                    showNotification('خطأ في استيراد الملف', 'error');
                }
            };
            reader.readAsArrayBuffer(file);
        }

        function saveDatabase() {
            db.lastModified = new Date().toISOString();
            localStorage.setItem('grad_db_shared', JSON.stringify(db));
        }

        function registerUser() {
            let users = JSON.parse(localStorage.getItem('unique_users')) || [];
            const userKey = 'user_' + Math.random().toString(36).substr(2, 9);
            
            if (!users.includes(userKey)) {
                users.push(userKey);
                localStorage.setItem('unique_users', JSON.stringify(users));
            }
        }

        function showNotification(message, type = 'info') {
            // إنشاء إشعار بسيط
            const notification = document.createElement('div');
            notification.className = `fixed top-4 right-4 px-6 py-3 rounded-xl text-white font-bold shadow-lg z-50 ${
                type === 'success' ? 'bg-emerald-600' : 
                type === 'error' ? 'bg-rose-600' : 
                type === 'warning' ? 'bg-amber-600' : 'bg-indigo-600'
            }`;
            notification.innerHTML = `
                <div class="flex items-center gap-3">
                    <i class="fas fa-${type === 'success' ? 'check-circle' : type === 'error' ? 'exclamation-circle' : 'info-circle'}"></i>
                    <span>${message}</span>
                </div>
            `;
            
            document.body.appendChild(notification);
            
            setTimeout(() => {
                notification.style.opacity = '0';
                setTimeout(() => {
                    document.body.removeChild(notification);
                }, 300);
            }, 3000);
        }

        // واتساب
        function showWhatsAppModal() {
            if (!currentProject) {
                alert('يرجى اختيار مشروع أولاً');
                return;
            }
            
            const modal = document.getElementById('whatsappModal');
            modal.style.display = 'flex';
            
            // تحديث معاينة الرسالة
            updateMessagePreview();
            
            // تحديث معاينة الرسالة عند التغيير
            document.getElementById('evaluatorName').addEventListener('input', updateMessagePreview);
            document.getElementById('customMessage').addEventListener('input', updateMessagePreview);
        }

        function closeWhatsAppModal() {
            document.getElementById('whatsappModal').style.display = 'none';
        }

        function updateMessagePreview() {
            const evaluatorName = document.getElementById('evaluatorName').value || 'المقيم';
            const customMessage = document.getElementById('customMessage').value;
            const project = currentProject;
            const roleName = roles[currentRole].roleName;
            const date = document.getElementById('evalDate').value;
            
            let message = `📋 *تقرير التقييم النهائي* 📋\n\n`;
            message += `🎓 *نوع التقييم:* ${roleName}\n`;
                message += `📁 *اسم المشروع:* ${project.title}\n`;
            message += `👨‍🏫 *المشرف الأكاديمي:* ${project.supervisor}\n`;
            message += `📅 *تاريخ التقييم:* ${date}\n\n`;
            message += `📊 *نتائج تقييم الطلاب:*\n`;
            message += `══════════════════\n\n`;
            
            document.querySelectorAll('.student-card').forEach((card, index) => {
                const studentName = card.querySelector('.student-name-display').textContent;
                const totalScore = card.querySelector('.student-total-display').textContent;
                const grade = card.querySelector('.student-result-text').textContent;
                
                message += `👤 *الطالب ${index + 1}:* ${studentName}\n`;
                message += `⭐ *الدرجة النهائية:* ${totalScore}/100\n`;
                message += `🏆 *التقدير:* ${grade}\n`;
                message += `────────────────────\n`;
            });
            
            message += `\n${customMessage}\n\n`;
            message += `🔸 *ملاحظة:* هذا التقرير تم إنشاؤه تلقائياً بواسطة نظام تقييم مشاريع التخرج`;
            
            document.getElementById('messagePreview').textContent = message;
        }

        function sendWhatsAppMessage() {
            const evaluatorName = document.getElementById('evaluatorName').value || 'المقيم';
            const phoneNumber = document.getElementById('phoneNumber').value;
            const project = currentProject;
            const roleName = roles[currentRole].roleName;
            const date = document.getElementById('evalDate').value;
            
            let message = `📋 *تقرير التقييم النهائي* 📋%0A%0A`;
            message += `🎓 *نوع التقييم:* ${roleName}%0A`;
            message += `👨‍🏫 *اسم المقيم:* ${evaluatorName}%0A`;
            message += `📁 *اسم المشروع:* ${project.title}%0A`;
            message += `👨‍🏫 *المشرف الأكاديمي:* ${project.supervisor}%0A`;
            message += `📅 *تاريخ التقييم:* ${date}%0A%0A`;
            message += `📊 *نتائج تقييم الطلاب:*%0A`;
            message += `══════════════════%0A%0A`;
            
            document.querySelectorAll('.student-card').forEach((card, index) => {
                const studentName = card.querySelector('.student-name-display').textContent;
                const totalScore = card.querySelector('.student-total-display').textContent;
                const grade = card.querySelector('.student-result-text').textContent;
                
                message += `👤 *الطالب ${index + 1}:* ${studentName}%0A`;
                message += `⭐ *الدرجة النهائية:* ${totalScore}/100%0A`;
                message += `🏆 *التقدير:* ${grade}%0A`;
                message += `────────────────────%0A`;
            });
            
            const customMessage = document.getElementById('customMessage').value;
            if (customMessage.trim()) {
                message += `%0A${customMessage}%0A%0A`;
            }
            
            message += `🔸 *ملاحظة:* هذا التقرير تم إنشاؤه تلقائياً بواسطة نظام تقييم مشاريع التخرج`;
            
            let whatsappURL = `https://wa.me/?text=${message}`;
            
            // إذا كان هناك رقم هاتف، إرسال له مباشرة
            if (phoneNumber.trim()) {
                const cleanNumber = phoneNumber.replace(/\D/g, '');
                whatsappURL = `https://wa.me/${cleanNumber}?text=${message}`;
            }
            
            window.open(whatsappURL, '_blank');
            closeWhatsAppModal();
        }

        // التصدير
        function exportToExcel() {
            if (!currentProject) {
                alert('يرجى اختيار مشروع أولاً');
                return;
            }
            
            const project = currentProject;
            const roleName = roles[currentRole].roleName;
            const date = document.getElementById('evalDate').value;
            
            const data = [
                ["تقرير نتائج التقييم النهائي"],
                ["المشروع", project.title],
                ["المشرف الأكاديمي", project.supervisor],
                ["تاريخ التقييم", date],
                [],
                ["م", "اسم الطالب", "الدرجة النهائية (من 100)", "التقدير"]
            ];
            
            document.querySelectorAll('.student-card').forEach((card, index) => {
                data.push([
                    index + 1,
                    card.querySelector('.student-name-display').innerText,
                    card.querySelector('.student-total-display').innerText,
                    card.querySelector('.student-result-text').innerText
                ]);
            });
            
            const ws = XLSX.utils.aoa_to_sheet(data);
            const wb = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(wb, ws, "النتائج");
            XLSX.writeFile(wb, `تقرير_تقييم_${project.title}_${new Date().toISOString().split('T')[0]}.xlsx`);
            
            showNotification('تم تصدير التقرير إلى Excel', 'success');
        }

        // تهيئة النظام
        document.addEventListener('DOMContentLoaded', function() {
            updateStats();
            console.log('نظام تقييم مشاريع التخرج محمل وجاهز للاستخدام');
        });
    </script>
</body>
</html>
