<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ZUS Staff Management</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f7f9fc;
        }
        .zus-blue { background-color: #007bff; }
        .zus-text-blue { color: #007bff; }
        .zus-yellow { background-color: #ffc107; }
        .zus-shadow { box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05); }
        .card { background-color: white; border-radius: 12px; padding: 1.5rem; }
        .input-field {
            width: 100%; 
            padding: 12px; 
            border: 1px solid #d1d5db; 
            border-radius: 8px; 
            font-size: 1rem;
            transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
        }
        .input-field:focus {
            border-color: #007bff;
            box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.25);
            outline: none;
        }
        .staff-list-item {
            display: grid;
            grid-template-columns: 1fr 100px 100px 50px; /* Name, Type, Rate, Actions */
            gap: 1rem;
            align-items: center;
        }
        @media (max-width: 640px) {
            .staff-list-item {
                grid-template-columns: 1fr 100px 50px; /* Adjust for mobile */
            }
            .staff-list-item .hidden-mobile {
                display: none;
            }
        }

        /* General Modal Styling */
        .modal-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.6);
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 50;
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.3s;
        }
        .modal-overlay.active {
            opacity: 1;
            pointer-events: auto;
        }
        .modal-content {
            background: white;
            border-radius: 12px;
            padding: 2rem;
            width: 90%;
            max-width: 400px;
            transform: translateY(50px);
            opacity: 0;
            transition: transform 0.3s ease-out, opacity 0.3s ease-out;
        }
        .modal-overlay.active .modal-content {
            transform: translateY(0);
            opacity: 1;
        }
    </style>
</head>
<body class="p-4 sm:p-8">

    <div id="app-container" class="max-w-5xl mx-auto">
        <!-- Header -->
        <header class="text-center mb-6">
            <h1 class="text-3xl sm:text-4xl font-extrabold text-gray-800">Staff & Roster Management</h1>
            <p class="text-lg text-gray-500 mt-2">Maintain your staff list and availability for easy scheduling.</p>
            <p id="status-message" class="mt-2 text-sm text-green-600 font-medium"></p>
        </header>
        
        <!-- Navigation -->
        <div class="card zus-shadow p-4 mb-8 flex justify-between items-center">
            <a href="index.html" class="text-sm font-semibold zus-text-blue hover:underline">
                &larr; Go To Leave Calendar
            </a>
            <a href="cost_prediction.html" class="text-sm font-semibold zus-text-blue hover:underline">
                Go To Cost Prediction &rarr;
            </a>
        </div>

        <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
            <!-- 1. Add Staff Member Form -->
            <div class="card zus-shadow lg:col-span-1 h-fit">
                <h2 class="text-2xl font-bold text-gray-800 mb-4">Add New Staff</h2>
                
                <form id="add-staff-form" onsubmit="handleAddStaff(event)">
                    <div class="mb-4">
                        <label for="staff-name" class="block text-sm font-medium text-gray-700 mb-1">Full Name</label>
                        <input type="text" id="staff-name" required placeholder="e.g., John Doe" class="input-field">
                    </div>
                    
                    <div class="mb-4">
                        <label for="employment-type" class="block text-sm font-medium text-gray-700 mb-1">Employment Type</label>
                        <select id="employment-type" required class="input-field bg-white">
                            <option value="Part-Time">Part-Time (Hourly)</option>
                            <option value="Full-Time">Full-Time (Fixed Salary)</option>
                        </select>
                    </div>

                    <div class="mb-4">
                        <label for="hourly-rate" class="block text-sm font-medium text-gray-700 mb-1">Hourly Rate (RM)</label>
                        <input type="number" id="hourly-rate" required placeholder="e.g., 10.00 (For PT)" min="0" step="0.01" value="10.00" class="input-field">
                        <p class="text-xs text-gray-500 mt-1">Only relevant for Part-Time staff.</p>
                    </div>

                    <button type="submit" class="w-full zus-blue text-white font-semibold py-3 rounded-lg hover:opacity-90 transition duration-150 mt-4">
                        <span id="add-staff-button-text">Add Staff Member</span>
                    </button>
                </form>
            </div>

            <!-- 2. Staff List Display -->
            <div class="card zus-shadow lg:col-span-2">
                <h2 class="text-2xl font-bold text-gray-800 mb-4">Current Staff Roster (<span id="staff-count">0</span>)</h2>
                
                <div class="staff-list-item text-sm font-bold text-gray-600 border-b pb-2 mb-3 hidden sm:grid">
                    <div>Name / ID</div>
                    <div class="hidden-mobile">Type</div>
                    <div>Rate (RM/hr)</div>
                    <div>Actions</div>
                </div>

                <div id="staff-list" class="space-y-3">
                    <p class="text-gray-500 text-center py-4">Use the form on the left to add your first staff member.</p>
                </div>
            </div>
        </div>

        <div class="card zus-shadow mt-8">
             <h2 class="text-2xl font-bold text-gray-800 mb-4">Next Step: Scheduling</h2>
             <p class="text-gray-600">Once you have added all your staff here, you can proceed to the Shift Scheduler page to start building your weekly roster based on availability and cost targets.</p>
             <a href="shift_scheduler.html" class="inline-block mt-4 zus-yellow text-gray-900 font-bold py-2 px-4 rounded-lg hover:opacity-90 transition duration-150">
                 Go to Shift Scheduler &rarr;
             </a>
        </div>
    </div>
    
    <!-- Custom Modal for Alerts (replacing native alert/confirm) -->
    <div id="custom-alert-modal" class="modal-overlay">
        <div id="alert-modal-content" class="modal-content">
            <h3 class="text-xl font-bold mb-3 text-gray-800" id="alert-title"></h3>
            <p class="text-gray-600 mb-6" id="alert-message"></p>
            <button id="alert-close" class="zus-blue text-white w-full py-2 rounded-lg font-semibold hover:opacity-90">OK</button>
        </div>
    </div>
    
    <!-- Firebase SDKs -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, collection, addDoc, onSnapshot, query, deleteDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Set Firestore log level for debugging
        setLogLevel('Debug');

        // --- GLOBAL VARIABLES (Provided by Canvas Environment) ---
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : null;
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
        
        // --- FIREBASE INITIALIZATION ---
        let db;
        let auth;
        let userId = null;
        let isAuthReady = false;
        
        // --- STAFF DATA ARRAY ---
        let staffList = [];

        if (firebaseConfig) {
            try {
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);

                onAuthStateChanged(auth, async (user) => {
                    if (user) {
                        userId = user.uid;
                        isAuthReady = true;
                        document.getElementById('status-message').textContent = `Connected and syncing as User ID: ${userId}`;
                        loadStaffData();
                    } else {
                        try {
                            if (initialAuthToken) {
                                await signInWithCustomToken(auth, initialAuthToken);
                            } else {
                                await signInAnonymously(auth);
                            }
                        } catch (error) {
                            console.error("Authentication failed:", error);
                            document.getElementById('status-message').textContent = 'Authentication failed. Data may not save.';
                            isAuthReady = true;
                        }
                    }
                });
            } catch (e) {
                console.error("Firebase Initialization Error:", e);
                document.getElementById('status-message').textContent = 'Error initializing Firebase.';
            }
        } else {
            document.getElementById('status-message').textContent = 'Firebase config missing. Staff data cannot be saved.';
        }

        // --- FIRESTORE UTILITIES ---
        const getStaffCollectionRef = () => {
            if (!db || !userId) return null;
            // Private collection for staff list (tied to the specific user/store)
            return collection(db, `artifacts/${appId}/users/${userId}/staff_roster`);
        };
        
        // --- CRUD OPERATIONS ---

        const loadStaffData = () => {
            if (!isAuthReady || !db || !userId) return;

            const staffColRef = getStaffCollectionRef();
            if (!staffColRef) return;

            // Real-time listener for the staff roster
            onSnapshot(staffColRef, (querySnapshot) => {
                staffList = [];
                querySnapshot.forEach((doc) => {
                    staffList.push({ id: doc.id, ...doc.data() });
                });
                renderStaffList();
            }, (error) => {
                console.error("Error listening to staff roster:", error);
            });
        };
        
        window.handleAddStaff = async (e) => {
            e.preventDefault();
            if (!isAuthReady) {
                alertModal("Error", "System not authenticated. Cannot save data.");
                return;
            }

            const name = document.getElementById('staff-name').value.trim();
            const type = document.getElementById('employment-type').value;
            let rate = parseFloat(document.getElementById('hourly-rate').value);

            if (!name) return;
            if (type === 'Part-Time' && (isNaN(rate) || rate <= 0)) {
                alertModal("Invalid Rate", "Part-Time staff must have a valid hourly rate greater than 0.");
                return;
            }
            // Full-Time staff rate is irrelevant for hourly calculations, set to 0
            if (type === 'Full-Time') {
                rate = 0; 
            }
            
            const staffColRef = getStaffCollectionRef();
            if (!staffColRef) return;
            
            document.getElementById('add-staff-button-text').textContent = 'Adding...';
            
            try {
                await addDoc(staffColRef, {
                    name: name,
                    type: type,
                    hourlyRate: rate,
                    recordedBy: userId,
                    timestamp: new Date().toISOString()
                });
                
                alertModal("Success", `${name} (${type}) added to the roster.`);
                document.getElementById('add-staff-form').reset();
            } catch (error) {
                console.error("Error saving staff data:", error);
                alertModal("Error", `Failed to add staff member. Check console for details.`);
            } finally {
                document.getElementById('add-staff-button-text').textContent = 'Add Staff Member';
            }
        };

        window.handleDeleteStaff = async (id, name) => {
            if (!isAuthReady) {
                alertModal("Error", "System not authenticated. Cannot perform action.");
                return;
            }

            // Custom confirmation logic (instead of native confirm())
            const modal = document.getElementById('custom-alert-modal');
            modal.classList.add('active');
            modal.querySelector('#alert-title').textContent = 'Confirm Deletion';
            modal.querySelector('#alert-message').textContent = `Are you sure you want to remove ${name} from the roster? This cannot be undone.`;
            
            const confirmButton = document.createElement('button');
            confirmButton.textContent = 'Yes, Delete';
            confirmButton.className = 'bg-red-500 text-white w-full py-2 rounded-lg font-semibold hover:bg-red-600 transition duration-150 mt-2';

            const closeButton = modal.querySelector('#alert-close');
            closeButton.textContent = 'Cancel';

            // Clear existing listeners
            const oldConfirm = document.getElementById('custom-confirm-button');
            if(oldConfirm) oldConfirm.remove();

            // Insert custom confirm button
            modal.querySelector('#alert-modal-content').appendChild(confirmButton);
            confirmButton.id = 'custom-confirm-button';


            confirmButton.onclick = async () => {
                confirmButton.disabled = true;
                const staffColRef = getStaffCollectionRef();
                if (!staffColRef) return;

                try {
                    await deleteDoc(doc(staffColRef, id));
                    alertModal("Deleted", `${name} has been removed.`);
                } catch (error) {
                    console.error("Error deleting staff:", error);
                    alertModal("Error", `Failed to delete ${name}. Check console for details.`);
                } finally {
                    modal.classList.remove('active');
                    confirmButton.remove(); // Clean up custom button
                    closeButton.textContent = 'OK'; // Restore default button text
                }
            };
            
            closeButton.onclick = () => {
                modal.classList.remove('active');
                const oldConfirm = document.getElementById('custom-confirm-button');
                if(oldConfirm) oldConfirm.remove();
                closeButton.textContent = 'OK'; // Restore default button text
            };
        };

        // --- RENDERING ---

        const renderStaffList = () => {
            const staffListEl = document.getElementById('staff-list');
            const staffCountEl = document.getElementById('staff-count');
            staffListEl.innerHTML = '';
            staffCountEl.textContent = staffList.length;

            if (staffList.length === 0) {
                staffListEl.innerHTML = '<p class="text-gray-500 text-center py-4">Use the form on the left to add your first staff member.</p>';
                return;
            }

            staffList.sort((a, b) => a.name.localeCompare(b.name));

            staffList.forEach(staff => {
                const rateDisplay = staff.type === 'Part-Time' ? staff.hourlyRate.toFixed(2) : '--';
                const staffItem = document.createElement('div');
                staffItem.className = 'staff-list-item border-b border-gray-100 py-3 hover:bg-gray-50 transition duration-100 rounded-md px-2';
                staffItem.innerHTML = `
                    <div class="truncate">
                        <span class="font-semibold text-gray-800">${staff.name}</span>
                        <p class="text-xs text-gray-400 mt-0.5 sm:hidden">ID: ${staff.id.substring(0, 4)}...</p>
                        <p class="text-xs text-gray-400 mt-0.5 hidden sm:block">ID: ${staff.id}</p>
                    </div>
                    <div class="font-medium text-sm text-gray-600 hidden-mobile">${staff.type}</div>
                    <div class="font-medium text-sm text-gray-700">RM ${rateDisplay}</div>
                    <div>
                        <button onclick="handleDeleteStaff('${staff.id}', '${staff.name}')" 
                                class="text-red-500 hover:text-red-700 text-lg transition duration-150" 
                                title="Remove Staff">
                            &times;
                        </button>
                    </div>
                `;
                staffListEl.appendChild(staffItem);
            });
        };
        
        // --- Custom Alert/Confirmation System (Replacing native functions) ---
        const createModalBase = (id) => {
            let modal = document.getElementById(id);
            if (!modal) {
                modal = document.createElement('div');
                modal.id = id;
                modal.className = 'fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-[101] hidden';
                document.body.appendChild(modal);
            }
            return modal;
        };

        const alertModal = (title, message) => {
            const modal = document.getElementById('custom-alert-modal');
            
            // Cleanup in case a previous confirmation was interrupted
            const oldConfirm = document.getElementById('custom-confirm-button');
            if(oldConfirm) oldConfirm.remove();
            
            modal.querySelector('#alert-title').textContent = title;
            modal.querySelector('#alert-message').textContent = message;
            modal.querySelector('#alert-close').textContent = 'OK';
            
            // Show modal
            modal.classList.add('active');
            
            // Fade in content
            const modalContent = modal.querySelector('#alert-modal-content');
            modalContent.classList.remove('scale-95', 'opacity-0');
            setTimeout(() => {
                modalContent.style.transform = 'translateY(0)';
                modalContent.style.opacity = '1';
            }, 10);


            const closeModal = () => {
                // Fade out content
                modalContent.style.transform = 'translateY(50px)';
                modalContent.style.opacity = '0';
                
                // Hide modal overlay
                modal.classList.remove('active');
            };

            modal.querySelector('#alert-close').onclick = closeModal;
        };

        // Initial check for staff data load
        document.addEventListener('DOMContentLoaded', () => {
            // Placeholder: Check if authentication is already complete
            if (isAuthReady) {
                loadStaffData();
            }
        });
    </script>
</body>
</html>
