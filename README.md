# NET.ATTENDANCE
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Academic Attendance Management System</title>
    <!-- Tailwind CSS for modern UI styling -->
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
    </style>
</head>
<body class="bg-gray-100 min-h-screen p-6">

    <div class="max-w-6xl mx-auto bg-white rounded-xl shadow-md overflow-hidden p-6 mt-6">
        <div class="text-center border-b pb-4 mb-6">
            <h1 class="text-3xl font-bold text-gray-800">Academic Attendance Portal</h1>
            <p class="text-gray-500 mt-1">Track and manage course attendance records across semesters</p>
        </div>

        <!-- Input Form Section -->
        <div class="bg-gray-50 p-4 rounded-lg border border-gray-200 mb-6">
            <h2 class="text-lg font-semibold text-gray-700 mb-3">Mark New Attendance</h2>
            <form id="attendanceForm" class="grid grid-cols-1 md:grid-cols-3 lg:grid-cols-5 gap-4 items-end">
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-1">Student/Staff ID</label>
                    <input type="text" id="memberId" required class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 bg-white" placeholder="e.g. STU101">
                </div>
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-1">Full Name</label>
                    <input type="text" id="memberName" required class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 bg-white" placeholder="John Doe">
                </div>
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-1">Course</label>
                    <select id="courseSelect" required class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 bg-white">
                        <option value="" disabled selected>Select a Course</option>
                        <option value="Algebra & Calculus">Algebra & Calculus</option>
                        <option value="IT Essentials">IT Essentials</option>
                        <option value="Computer Hardware">Computer Hardware</option>
                        <option value="Business Management">Business Management</option>
                        <option value="Programming with Visual Basic.NET">Programming with Visual Basic.NET</option>
                    </select>
                </div>
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-1">Academic Week</label>
                    <select id="weekSelect" required class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 bg-white">
                        <option value="" disabled selected>Select Week</option>
                        <!-- Programmatically populated via JavaScript loop to keep code clean -->
                    </select>
                </div>
                <div>
                    <button type="submit" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-medium py-2 px-4 rounded-md transition duration-200 cursor-pointer">
                        Check In
                    </button>
                </div>
            </form>
        </div>

        <!-- Data Display Section -->
        <div>
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-xl font-bold text-gray-800">Attendance Log</h2>
                <button id="clearBtn" class="bg-red-500 hover:bg-red-600 text-white text-sm font-medium py-1.5 px-3 rounded-md transition duration-200 cursor-pointer">
                    Clear All Logs
                </button>
            </div>

            <div class="overflow-x-auto border border-gray-200 rounded-lg">
                <table class="min-w-full divide-y divide-gray-200">
                    <thead class="bg-gray-50">
                        <tr>
                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">ID</th>
                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Name</th>
                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Course</th>
                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Timeline</th>
                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Timestamp</th>
                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Status</th>
                        </tr>
                    </thead>
                    <tbody id="attendanceTableBody" class="bg-white divide-y divide-gray-200">
                        <!-- Content inserted dynamically by JavaScript -->
                    </tbody>
                </table>
            </div>
            
            <div id="noDataMessage" class="text-center text-gray-500 py-8 hidden">
                No attendance recorded yet.
            </div>
        </div>
    </div>

    <!-- Core Programming Logic -->
    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const attendanceForm = document.getElementById('attendanceForm');
            const memberIdInput = document.getElementById('memberId');
            const memberNameInput = document.getElementById('memberName');
            const courseSelect = document.getElementById('courseSelect');
            const weekSelect = document.getElementById('weekSelect');
            const tableBody = document.getElementById('attendanceTableBody');
            const clearBtn = document.getElementById('clearBtn');
            const noDataMessage = document.getElementById('noDataMessage');

            // Populate the Week selector from Week 1 to Week 19 programmatically
            for (let i = 1; i <= 19; i++) {
                const option = document.createElement('option');
                option.value = `Week ${i}`;
                option.textContent = `Week ${i}`;
                weekSelect.appendChild(option);
            }

            // 1. Retrieve records from LocalStorage
            let records = JSON.parse(localStorage.getItem('academicAttendanceRecords')) || [];

            // 2. Render data on initial page load
            renderTable();

            // 3. Form Submit Event Handler
            attendanceForm.addEventListener('submit', (e) => {
                e.preventDefault();

                const now = new Date();
                const dateString = now.toLocaleDateString(undefined, { year: 'numeric', month: 'short', day: 'numeric' });
                const timeString = now.toLocaleTimeString(undefined, { hour: '2-digit', minute: '2-digit' });

                // Construct expanded log object
                const newRecord = {
                    id: memberIdInput.value.trim(),
                    name: memberNameInput.value.trim(),
                    course: courseSelect.value,
                    week: weekSelect.value,
                    timestamp: `${dateString} @ ${timeString}`,
                    status: 'Present'
                };

                // Add record to state array (top layout position)
                records.unshift(newRecord);
                localStorage.setItem('academicAttendanceRecords', JSON.stringify(records));

                // Reset form inputs and refresh view
                attendanceForm.reset();
                renderTable();
            });

            // 4. Clear data helper
            clearBtn.addEventListener('click', () => {
                if (confirm('Are you sure you want to permanently delete all attendance logs?')) {
                    records = [];
                    localStorage.removeItem('academicAttendanceRecords');
                    renderTable();
                }
            });

            // 5. DOM Manipulation / Rendering Logic
            function renderTable() {
                tableBody.innerHTML = '';

                if (records.length === 0) {
                    noDataMessage.classList.remove('hidden');
                    return;
                } else {
                    noDataMessage.classList.add('hidden');
                }

                records.forEach(record => {
                    const row = document.createElement('tr');
                    row.className = "hover:bg-gray-50 transition-colors";
                    row.innerHTML = `
                        <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">${escapeHtml(record.id)}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-600">${escapeHtml(record.name)}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-indigo-600">${record.course}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                            <span class="px-2 py-1 bg-gray-200 text-gray-800 text-xs font-semibold rounded">
                                ${record.week}
                            </span>
                        </td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-400">${record.timestamp}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm">
                            <span class="px-2.5 py-0.5 inline-flex text-xs leading-5 font-semibold rounded-full bg-green-100 text-green-800">
                                ${record.status}
                            </span>
                        </td>
                    `;
                    tableBody.appendChild(row);
                });
            }

            // Security sanitization utility against HTML Injection
            function escapeHtml(string) {
                const map = { '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#039;' };
                return string.replace(/[&<>"']/g, function(m) { return map[m]; });
            }
        });
    </script>
</body>
</html>
