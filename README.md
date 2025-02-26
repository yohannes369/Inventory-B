# Inventory-B mnagement system
CREATE TABLE employees (
    employee_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    date_of_birth DATE,
    role ENUM('Admin', 'Manager', 'Staff', 'Stock Clerk', 'DIN', 'Head') NOT NULL,
    office_number VARCHAR(20),
    phone_number VARCHAR(20) NOT NULL,
    address TEXT,
    hire_date DATE DEFAULT CURRENT_DATE,
    status ENUM('Active', 'Inactive') DEFAULT 'Active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);



const express = require("express");
const bcrypt = require("bcryptjs");
const jwt = require("jsonwebtoken");
const db = require("../config/db"); // MySQL Connection
const router = express.Router();

// Register Employee
router.post("/register", async (req, res) => {
  const { first_name, last_name, email, password, role, phone_number } = req.body;

  const hashedPassword = await bcrypt.hash(password, 10);
  const sql = "INSERT INTO employees (first_name, last_name, email, password_hash, role, phone_number) VALUES (?, ?, ?, ?, ?, ?)";

  db.query(sql, [first_name, last_name, email, hashedPassword, role, phone_number], (err, result) => {
    if (err) return res.status(500).json({ error: err.message });
    res.json({ message: "Employee Registered Successfully!" });
  });
});

// Login Employee
router.post("/login", (req, res) => {
  const { email, password } = req.body;
  const sql = "SELECT * FROM employees WHERE email = ?";

  db.query(sql, [email], async (err, results) => {
    if (err) return res.status(500).json({ error: err.message });
    if (results.length === 0) return res.status(401).json({ error: "Invalid email or password" });

    const validPassword = await bcrypt.compare(password, results[0].password_hash);
    if (!validPassword) return res.status(401).json({ error: "Invalid email or password" });

    const token = jwt.sign({ employee_id: results[0].employee_id, role: results[0].role }, process.env.JWT_SECRET, { expiresIn: "1h" });
    res.json({ message: "Login successful!", token });
  });
});

// Get All Employees
router.get("/", (req, res) => {
  db.query("SELECT employee_id, first_name, last_name, email, role, phone_number FROM employees", (err, results) => {
    if (err) return res.status(500).json({ error: err.message });
    res.json(results);
  });
});

// Export Routes
module.exports = router;






📂 inventory-system/
├── 📂 src/
│   ├── 📂 config/         # Database connection & configurations
│   │   ├── db.js         # MySQL connection setup
│   │   ├── env.js        # Environment variable handler
│   │
│   ├── 📂 controllers/   # Business logic for handling API requests
│   │   ├── authController.js       # Authentication (Login, Register)
│   │   ├── userController.js       # Admin User Management (CRUD)
│   │   ├── inventoryController.js  # Stock Clerk - Add, Delete, Update Items
│   │   ├── requestController.js    # Staff - Request Items
│   │   ├── departmentController.js # DIN & Department - Accept/Reject Requests
│   │   ├── reportController.js     # Manager - View Reports & Comment
│   │
│   ├── 📂 middleware/    # Middleware functions (auth, error handling)
│   │   ├── authMiddleware.js   # JWT authentication & Role-based access
│   │   ├── errorMiddleware.js  # Global error handler
│   │
│   ├── 📂 models/        # Database schemas (Using SQL queries)
│   │   ├── userModel.js      # User database queries
│   │   ├── inventoryModel.js # Inventory database queries
│   │   ├── requestModel.js   # Item request database queries
│   │
│   ├── 📂 routes/        # API routes
│   │   ├── authRoutes.js       # Routes for login & authentication
│   │   ├── userRoutes.js       # Routes for Admin user management
│   │   ├── inventoryRoutes.js  # Routes for Stock Clerk inventory management
│   │   ├── requestRoutes.js    # Routes for Staff item requests
│   │   ├── departmentRoutes.js # Routes for DIN & Department approval
│   │   ├── reportRoutes.js     # Routes for Manager reports & comments
│   │
│   ├── 📂 utils/         # Utility functions (helpers)
│   │   ├── hashPassword.js  # Password hashing function
│   │   ├── jwtUtils.js      # JWT token functions
│   │
│   ├── app.js            # Express app initialization
│   ├── server.js         # Main entry point (starts the server)
│
├── 📂 tests/             # Unit & integration tests
│   ├── user.test.js      # Test cases for user module
│   ├── inventory.test.js # Test cases for inventory module
│
├── .env                  # Environment variables
├── .gitignore            # Git ignore file
├── package.json          # Node.js dependencies & scripts
├── README.md             # Documentation
