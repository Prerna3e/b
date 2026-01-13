MULTER
................................................................................................................................................................
Single File Upload (Image Upload)
// server.js
const express = require("express");
const multer = require("multer");
const path = require("path");

const app = express();

// Storage config
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, "uploads/");
  },
  filename: (req, file, cb) => {
    cb(null, Date.now() + path.extname(file.originalname));
  }
});

// File filter
const fileFilter = (req, file, cb) => {
  const allowed = /jpg|jpeg|png/;
  const ext = path.extname(file.originalname).toLowerCase();
  if (allowed.test(ext)) cb(null, true);
  else cb(new Error("Only .jpg and .png files are allowed"));
};

const upload = multer({ storage, fileFilter });

// Upload route
app.post("/upload", upload.single("profile"), (req, res) => {
  res.json({
    message: "File uploaded successfully",
    filename: req.file.filename,
    path: req.file.path
  });
});

// Error handler
app.use((err, req, res, next) => {
  res.status(400).json({ error: err.message });
});

app.listen(3000);
..................................................................................................................................................................
Multiple file upload
// server.js
const express = require("express");
const multer = require("multer");
const path = require("path");

const app = express();

// Storage config
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, "uploads/gallery");
  },
  filename: (req, file, cb) => {
    cb(null, Date.now() + "-" + file.originalname);
  }
});

// File filter
const fileFilter = (req, file, cb) => {
  const allowed = /jpg|jpeg|png/;
  const ext = path.extname(file.originalname).toLowerCase();
  if (allowed.test(ext)) cb(null, true);
  else cb(new Error("Only jpg, jpeg, png files allowed"));
};

// Multer config (max 5 files)
const upload = multer({
  storage,
  fileFilter,
  limits: { files: 5 }
});

// Upload route
app.post("/upload-multiple", upload.array("images", 5), (req, res) => {
  const files = req.files.map(f => f.filename);
  res.json({
    message: "Files uploaded successfully",
    files
  });
});

// Error handler
app.use((err, req, res, next) => {
  res.status(400).json({ error: err.message });
});

app.listen(3000);
.................................................................................................................................................................
File Upload with Form Data
// server.js
const express = require("express");
const multer = require("multer");
const path = require("path");
const fs = require("fs");

const app = express();
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Storage
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, "uploads/resumes");
  },
  filename: (req, file, cb) => {
    cb(null, Date.now() + path.extname(file.originalname));
  }
});

// File filter (PDF only)
const fileFilter = (req, file, cb) => {
  if (path.extname(file.originalname).toLowerCase() === ".pdf")
    cb(null, true);
  else cb(new Error("Only PDF files allowed"));
};

const upload = multer({ storage, fileFilter });

// Upload route
app.post("/submit", upload.single("resume"), (req, res) => {
  const { name, email, phone } = req.body;

  const data = {
    name,
    email,
    phone,
    resumePath: req.file.path
  };

  let records = [];
  if (fs.existsSync("data.json")) {
    records = JSON.parse(fs.readFileSync("data.json"));
  }

  records.push(data);
  fs.writeFileSync("data.json", JSON.stringify(records, null, 2));

  res.json({
    message: "Form submitted successfully",
    data
  });
});

// Error handler
app.use((err, req, res, next) => {
  res.status(400).json({ error: err.message });
});

app.listen(3000);
.................................................................................................................................................................
File Size & Type Validation
// server.js
const express = require("express");
const multer = require("multer");
const path = require("path");

const app = express();

// Storage configuration
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, "uploads/products");
  },
  filename: (req, file, cb) => {
    cb(null, Date.now() + path.extname(file.originalname));
  }
});

// File type validation
const fileFilter = (req, file, cb) => {
  const allowedTypes = /jpg|png/;
  const ext = path.extname(file.originalname).toLowerCase();
  if (allowedTypes.test(ext)) cb(null, true);
  else cb(new Error("Only .jpg and .png files are allowed"));
};

// Multer config (2 MB limit)
const upload = multer({
  storage,
  fileFilter,
  limits: { fileSize: 2 * 1024 * 1024 }
});

// Upload route
app.post("/upload-product", upload.single("product"), (req, res) => {
  res.json({
    message: "File uploaded successfully",
    path: req.file.path,
    size: req.file.size
  });
});

// Error handler
app.use((err, req, res, next) => {
  res.status(400).json({ error: err.message });
});

app.listen(3000);
.................................................................................................................................................................
Multer - File Management API
// server.js
const express = require("express");
const multer = require("multer");
const path = require("path");
const fs = require("fs");

const app = express();
app.use(express.json());

// Serve static files
app.use("/uploads", express.static("uploads"));

// Ensure base folders exist
["uploads/profile_pics", "uploads/documents", "uploads/others"].forEach(dir => {
  if (!fs.existsSync(dir)) fs.mkdirSync(dir, { recursive: true });
});

// Multer storage
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    const userId = req.params.userId;
    let folder = "others";

    if (file.fieldname === "profilePic") folder = "profile_pics";
    if (file.fieldname === "docs") folder = "documents";

    const dest = path.join("uploads", folder, userId);
    fs.mkdirSync(dest, { recursive: true });
    cb(null, dest);
  },
  filename: (req, file, cb) => {
    const userId = req.params.userId;
    cb(
      null,
      `${file.fieldname}-${userId}-${Date.now()}-${file.originalname}`
    );
  }
});

// File filter
const fileFilter = (req, file, cb) => {
  const allowed = /jpg|png|pdf|docx/;
  const ext = path.extname(file.originalname).toLowerCase();
  if (allowed.test(ext)) cb(null, true);
  else cb(new Error("Invalid file type"));
};

// Multer config
const upload = multer({
  storage,
  fileFilter,
  limits: { fileSize: 5 * 1024 * 1024 }
});

// Upload route
app.post(
  "/upload/:userId",
  upload.fields([
    { name: "profilePic", maxCount: 1 },
    { name: "docs", maxCount: 5 },
    { name: "others", maxCount: 5 }
  ]),
  (req, res) => {
    const uploaded = {
      profilePic: req.files.profilePic
        ? req.files.profilePic[0].path
        : null,
      docs: req.files.docs ? req.files.docs.map(f => f.path) : [],
      others: req.files.others ? req.files.others.map(f => f.path) : []
    };

    res.json({
      message: "Files uploaded successfully!",
      uploaded
    });
  }
);

// Get files by userId
app.get("/files/:userId", (req, res) => {
  const userId = req.params.userId;
  const baseDirs = ["profile_pics", "documents", "others"];
  let files = [];

  baseDirs.forEach(dir => {
    const folder = path.join("uploads", dir, userId);
    if (fs.existsSync(folder)) {
      fs.readdirSync(folder).forEach(f => {
        files.push(path.join(folder, f));
      });
    }
  });

  res.json({ userId, files });
});

// Delete file
app.delete("/delete/:userId/:filename", (req, res) => {
  const { userId, filename } = req.params;
  const baseDirs = ["profile_pics", "documents", "others"];
  let found = false;

  baseDirs.forEach(dir => {
    const filePath = path.join("uploads", dir, userId, filename);
    if (fs.existsSync(filePath)) {
      fs.unlinkSync(filePath);
      found = true;
    }
  });

  if (!found) return res.status(404).json({ message: "File not found" });

  res.json({ message: "File deleted successfully!" });
});

// Multer error handler
app.use((err, req, res, next) => {
  if (err instanceof multer.MulterError || err.message)
    return res.status(400).json({ error: err.message });
  next();
});

app.listen(3000);
.................................................................................................................................................................
HTTP
................................................................................................................................................................
HTTP Module - Create a Basic API with POST and GET
// server.js
const http = require("http");
const fs = require("fs");
const path = require("path");

const server = http.createServer((req, res) => {
  if (req.url.startsWith("/public/")) {
    const filePath = path.join(__dirname, req.url);

    fs.readFile(filePath, (err, data) => {
      if (err) {
        res.writeHead(404, { "Content-Type": "text/plain" });
        res.end("404 File Not Found");
      } else {
        const ext = path.extname(filePath);
        let contentType = "text/plain";

        if (ext === ".css") contentType = "text/css";
        if (ext === ".js") contentType = "application/javascript";
        if (ext === ".png") contentType = "image/png";
        if (ext === ".jpg" || ext === ".jpeg") contentType = "image/jpeg";

        res.writeHead(200, { "Content-Type": contentType });
        res.end(data);
      }
    });
  } else {
    res.writeHead(404, { "Content-Type": "text/plain" });
    res.end("404 Not Found");
  }
});

server.listen(3000);

project/
│
├── server.js
└── public/
    ├── style.css
    ├── script.js
    └── image.png
..................................................................................................................................................................
HTTP - Serve Files from a Public Directory 
// server.js
const http = require("http");
const fs = require("fs");
const path = require("path");

const server = http.createServer((req, res) => {
  if (req.url.startsWith("/public/")) {
    const filePath = path.join(__dirname, req.url);

    fs.readFile(filePath, (err, data) => {
      if (err) {
        res.writeHead(404, { "Content-Type": "text/plain" });
        res.end("404 File Not Found");
      } else {
        const ext = path.extname(filePath);
        let contentType = "text/plain";

        if (ext === ".css") contentType = "text/css";
        else if (ext === ".js") contentType = "application/javascript";
        else if (ext === ".png") contentType = "image/png";
        else if (ext === ".jpg" || ext === ".jpeg") contentType = "image/jpeg";

        res.writeHead(200, { "Content-Type": contentType });
        res.end(data);
      }
    });
  } else {
    res.writeHead(404, { "Content-Type": "text/plain" });
    res.end("404 Not Found");
  }
});

server.listen(3000);


project/
│
├── server.js
└── public/
    ├── style.css
    ├── script.js
    └── image.png
................................................................................................................................................................
HTTP Module - GET and POST Handling
// server.js
const http = require("http");
const querystring = require("querystring");

const server = http.createServer((req, res) => {
  if (req.method === "GET" && req.url === "/form") {
    res.writeHead(200, { "Content-Type": "text/html" });
    res.end(`
      <form method="POST" action="/form">
        <input type="text" name="name" placeholder="Name" required />
        <input type="email" name="email" placeholder="Email" required />
        <button type="submit">Submit</button>
      </form>
    `);
  }

  else if (req.method === "POST" && req.url === "/form") {
    let body = "";

    req.on("data", chunk => {
      body += chunk;
    });

    req.on("end", () => {
      const data = querystring.parse(body);
      res.writeHead(200, { "Content-Type": "text/plain" });
      res.end(`Thank you, ${data.name}. Your email is ${data.email}.`);
    });
  }

  else {
    res.writeHead(404, { "Content-Type": "text/plain" });
    res.end("404 Not Found");
  }
});

server.listen(3000);
.................................................................................................................................................................
HTTP Module - Query Parameter Handling 
// server.js
const http = require("http");
const url = require("url");

const server = http.createServer((req, res) => {
  const parsedUrl = url.parse(req.url, true);

  if (parsedUrl.pathname === "/greet") {
    const name = parsedUrl.query.name || "Guest";
    res.writeHead(200, { "Content-Type": "text/plain" });
    res.end(`Hello, ${name}!`);
  } else {
    res.writeHead(404, { "Content-Type": "text/plain" });
    res.end("404 Not Found");
  }
});

server.listen(3000);
................................................................................................................................................................
HTTP Module - Redirect Based on Time 
// server.js
const http = require("http");

const server = http.createServer((req, res) => {
  if (req.url === "/time-check") {
    const hour = new Date().getHours();

    if (hour < 12) {
      res.writeHead(302, { Location: "/morning" });
      res.end();
    } else {
      res.writeHead(302, { Location: "/evening" });
      res.end();
    }
  }

  else if (req.url === "/morning") {
    res.writeHead(200, { "Content-Type": "text/plain" });
    res.end("Good Morning!");
  }

  else if (req.url === "/evening") {
    res.writeHead(200, { "Content-Type": "text/plain" });
    res.end("Good Evening!");
  }

  else {
    res.writeHead(404, { "Content-Type": "text/plain" });
    res.end("404 Not Found");
  }
});

server.listen(3000);
................................................................................................................................................................
HTTP Module - Respond with Different Content Types
// server.js
const http = require("http");

const server = http.createServer((req, res) => {
  if (req.url === "/data.html") {
    res.writeHead(200, { "Content-Type": "text/html" });
    res.end("<h1>Hello from HTML</h1>");
  }

  else if (req.url === "/data.txt") {
    res.writeHead(200, { "Content-Type": "text/plain" });
    res.end("Hello from Text");
  }

  else if (req.url === "/data.json") {
    res.writeHead(200, { "Content-Type": "application/json" });
    res.end(JSON.stringify({ message: "Hello from JSON" }));
  }

  else {
    res.writeHead(404, { "Content-Type": "text/plain" });
    res.end("404 Not Found");
  }
});

server.listen(3000);
................................................................................................................................................................
HTTP Server 

// server-express.js (WITH EXPRESS)
const express = require("express");
const fs = require("fs");

const app = express();
const movies = JSON.parse(fs.readFileSync("movie.json"));

app.get("/movies", (req, res) => {
  const { genre, year } = req.query;

  if (!genre || !year) {
    return res.json({ message: "Both genre and year are required", data: [] });
  }

  const years = year.split(",").map(y => Number(y));

  const result = movies.filter(
    m =>
      m.genre.toLowerCase() === genre.toLowerCase() &&
      years.includes(m.year)
  );

  res.json(result);
});

app.listen(3000);
json
Copy code
// movie.json
[
  { "title": "Fast Chase", "genre": "Action", "year": 2020, "rating": 7.5 },
  { "title": "Mission Reloaded", "genre": "Action", "year": 2022, "rating": 8.1 },
  { "title": "Love in Paris", "genre": "Romance", "year": 2020, "rating": 6.8 },
  { "title": "Laugh Out Loud", "genre": "Comedy", "year": 2022, "rating": 7.2 },
  { "title": "Silent Whisper", "genre": "Thriller", "year": 2021, "rating": 8.3 },
  { "title": "Battlefield Glory", "genre": "Action", "year": 2021, "rating": 7.9 },
  { "title": "Eternal Love", "genre": "Romance", "year": 2022, "rating": 7.0 }
]
................................................................................................................................................................
EXPRESS
................................................................................................................................................................
Express - Modular User Routes
// routes/users.js
const express = require("express");
const router = express.Router();

const users = [
  { id: 1, name: "Rahul", email: "rahul@gmail.com" },
  { id: 2, name: "Priya", email: "priya@gmail.com" },
  { id: 3, name: "Aman", email: "aman@gmail.com" }
];

router.get("/users", (req, res) => {
  res.json(users);
});

router.get("/users/:id", (req, res) => {
  const user = users.find(u => u.id === Number(req.params.id));
  if (!user) return res.status(404).json({ message: "User not found" });
  res.json(user);
});

module.exports = router;
js
Copy code
// app.js
const express = require("express");
const userRoutes = require("./routes/users");

const app = express();

app.use("/api", userRoutes);

app.listen(3000);
................................................................................................................................................................
Express - Multi-Module Blog 
// routes/posts.js
const express = require("express");
const router = express.Router();

const posts = [
  { id: 1, title: "First Post", content: "Hello World" },
  { id: 2, title: "Second Post", content: "Blog Content" }
];

router.get("/posts", (req, res) => {
  res.json(posts);
});

router.get("/posts/:postId", (req, res) => {
  const post = posts.find(p => p.id === Number(req.params.postId));
  if (!post) return res.status(404).json({ message: "Post not found" });
  res.json(post);
});

module.exports = router;
js
Copy code
// routes/comments.js
const express = require("express");
const router = express.Router();

const comments = [
  { id: 1, postId: 1, text: "Nice post!" },
  { id: 2, postId: 2, text: "Good read" }
];

router.get("/comments", (req, res) => {
  res.json(comments);
});

router.get("/comments/:commentId", (req, res) => {
  const comment = comments.find(c => c.id === Number(req.params.commentId));
  if (!comment) return res.status(404).json({ message: "Comment not found" });
  res.json(comment);
});

module.exports = router;
js
Copy code
// app.js
const express = require("express");
const postsRoutes = require("./routes/posts");
const commentsRoutes = require("./routes/comments");

const app = express();

app.use("/api", postsRoutes);
app.use("/api", commentsRoutes);

app.listen(3000);
.................................................................................................................................................................
Express -Product Routes with Dynamic Parameters
// routes/products.js
const express = require("express");
const router = express.Router();

const products = [
  { id: 1, category: "electronics", name: "Mobile", price: 15000 },
  { id: 2, category: "electronics", name: "Laptop", price: 60000 },
  { id: 3, category: "clothing", name: "T-Shirt", price: 800 }
];

router.get("/products", (req, res) => {
  res.json(products);
});

router.get("/products/:category/:id", (req, res) => {
  const { category, id } = req.params;
  const product = products.find(
    p => p.category === category && p.id === Number(id)
  );

  if (!product)
    return res.status(404).json({ message: "Product not found" });

  res.json(product);
});

module.exports = router;
js
Copy code
// app.js
const express = require("express");
const productRoutes = require("./routes/products");

const app = express();

app.use("/api", productRoutes);

app.listen(3000);
.................................................................................................................................................................
MONGOOOSE
...................................................................................................................................................................
Mongoose - Employee Payroll System

📁 models/Employee.js
const mongoose = require("mongoose");

const employeeSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    minlength: 3,
    maxlength: 50
  },

  employeeId: {
    type: String,
    required: true,
    unique: true
  },

  departments: {
    type: [String],
    required: true
  },

  salary: {
    type: Number,
    required: true,
    validate: {
      validator: function (value) {
        return value > 10000 && value % 1000 === 0;
      },
      message: "Salary must be greater than 10000 and a multiple of 1000"
    }
  },

  joiningDate: {
    type: Date,
    default: Date.now
  }
});

// Pre-save hook
employeeSchema.pre("save", function (next) {
  this.name = this.name
    .split(" ")
    .map(word => word.charAt(0).toUpperCase() + word.slice(1))
    .join(" ");
  next();
});

module.exports = mongoose.model("Employee", employeeSchema);
📁 routes/employeeRoutes.js
const express = require("express");
const router = express.Router();
const Employee = require("../models/Employee");

// Add employee
router.post("/employees", async (req, res) => {
  try {
    const emp = new Employee(req.body);
    await emp.save();
    res.status(201).json(emp);
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});

// View all employees
router.get("/employees", async (req, res) => {
  const employees = await Employee.find();
  res.json(employees);
});

// Get employee by ID
router.get("/employees/:id", async (req, res) => {
  const emp = await Employee.findOne({ employeeId: req.params.id });
  if (!emp) return res.status(404).json({ message: "Employee not found" });
  res.json(emp);
});

// Update salary or departments
router.put("/employees/:id", async (req, res) => {
  try {
    const emp = await Employee.findOneAndUpdate(
      { employeeId: req.params.id },
      {
        salary: req.body.salary,
        departments: req.body.departments
      },
      { new: true, runValidators: true }
    );

    if (!emp) return res.status(404).json({ message: "Employee not found" });
    res.json(emp);
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});

// Delete employee
router.delete("/employees/:id", async (req, res) => {
  const emp = await Employee.findOneAndDelete({ employeeId: req.params.id });
  if (!emp) return res.status(404).json({ message: "Employee not found" });
  res.json({ message: "Employee deleted successfully" });
});

// Bonus: Average salary by department
router.get("/employees/average/:dept", async (req, res) => {
  const result = await Employee.aggregate([
    { $match: { departments: req.params.dept } },
    { $group: { _id: null, avgSalary: { $avg: "$salary" } } }
  ]);

  if (result.length === 0)
    return res.json({ message: "No employees found" });

  res.json({ averageSalary: result[0].avgSalary });
});

module.exports = router;

📁 app.js
const express = require("express");
const mongoose = require("mongoose");
const employeeRoutes = require("./routes/employeeRoutes");

const app = express();
app.use(express.json());

mongoose.connect("mongodb://127.0.0.1:27017/payrollDB");

app.use("/api", employeeRoutes);

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
................................................................................................................................................................
mongoose - Event Management System
// models/Event.js
const mongoose = require("mongoose");

const eventSchema = new mongoose.Schema({
  eventName: {
    type: String,
    required: true,
    minlength: 3,
    maxlength: 50
  },
  date: {
    type: Date,
    required: true,
    validate: {
      validator: d => d > new Date()
    }
  },
  venue: {
    type: String,
    required: true
  },
  participants: {
    type: [String],
    validate: {
      validator: p => p.length > 0
    }
  },
  ticketPrice: {
    type: Number,
    required: true,
    validate: {
      validator: p => p > 0 && p < 10000
    }
  }
});

eventSchema.pre("save", function (next) {
  this.eventName = this.eventName
    .split(" ")
    .map(w => w.charAt(0).toUpperCase() + w.slice(1))
    .join(" ");
  next();
});

eventSchema.post("save", function () {
  console.log("Event added successfully.");
});

module.exports = mongoose.model("Event", eventSchema);
js
Copy c
// routes/eventRoutes.js
const express = require("express");
const router = express.Router();
const Event = require("../models/Event");

router.post("/events", async (req, res) => {
  try {
    const e = new Event(req.body);
    await e.save();
    res.json(e);
  } catch (err) {
    res.status(400).json(err.message);
  }
});

router.get("/events", async (req, res) => {
  res.json(await Event.find());
});

router.get("/events/:id", async (req, res) => {
  const e = await Event.findById(req.params.id);
  if (!e) return res.status(404).json("Not Found");
  res.json(e);
});

router.put("/events/:id", async (req, res) => {
  try {
    const e = await Event.findByIdAndUpdate(
      req.params.id,
      req.body,
      { new: true, runValidators: true }
    );
    if (!e) return res.status(404).json("Not Found");
    res.json(e);
  } catch (err) {
    res.status(400).json(err.message);
  }
});

router.delete("/events/:id", async (req, res) => {
  const e = await Event.findByIdAndDelete(req.params.id);
  if (!e) return res.status(404).json("Not Found");
  res.json("Deleted");
});

router.get("/events/venue/:venue", async (req, res) => {
  res.json(await Event.find({ venue: req.params.venue }));
});

module.exports = router;

// app.js
const express = require("express");
const mongoose = require("mongoose");
const eventRoutes = require("./routes/eventRoutes");

const app = express();
app.use(express.json());

mongoose.connect("mongodb://127.0.0.1:27017/eventDB");

app.use("/api", eventRoutes);

app.listen(3000);
..................................................................................................................................................................
MONGODB
..................................................................................................................................................................
MongoDb - Company Management System using MongoDB 
// app.js
const express = require("express");
const { MongoClient } = require("mongodb");

const app = express();
const url = "mongodb://127.0.0.1:27017";
const dbName = "CompanyDB";

let db, employees, departments;

MongoClient.connect(url).then(client => {
  db = client.db(dbName);
  employees = db.collection("employees");
  departments = db.collection("departments");

  employees.createIndex({ empid: 1 }, { unique: true });
  departments.createIndex({ deptid: 1 }, { unique: true });
});

// -------- EMPLOYEE ROUTES --------

app.get("/seed-employees", async (req, res) => {
  await employees.insertMany([
    { empid: 1, name: "Aman", salary: 30000, deptid: 101 },
    { empid: 2, name: "Riya", salary: 35000, deptid: 102 },
    { empid: 3, name: "Neha", salary: 40000, deptid: 101 },
    { empid: 4, name: "Raj", salary: 45000, deptid: 103 },
    { empid: 5, name: "Simran", salary: 38000, deptid: 102 },
    { empid: 6, name: "Kunal", salary: 42000, deptid: 103 }
  ]);
  res.send("Employees Seeded");
});

app.get("/view-employees", async (req, res) => {
  res.json(await employees.find().toArray());
});

app.get("/add-employee/:id/:name/:salary/:deptid", async (req, res) => {
  await employees.insertOne({
    empid: Number(req.params.id),
    name: req.params.name,
    salary: Number(req.params.salary),
    deptid: Number(req.params.deptid)
  });
  res.send("Employee Added");
});

app.get("/delete-employee/:id", async (req, res) => {
  await employees.deleteOne({ empid: Number(req.params.id) });
  res.send("Employee Deleted");
});

app.get("/delete-employee-lt/:id", async (req, res) => {
  await employees.deleteMany({ empid: { $lt: Number(req.params.id) } });
  res.send("Employees Deleted");
});

app.get("/update-salary/:id/:amount", async (req, res) => {
  await employees.updateOne(
    { empid: Number(req.params.id) },
    { $set: { salary: Number(req.params.amount) } }
  );
  res.send("Salary Updated");
});

app.get("/update-salary-range/:id/:amount", async (req, res) => {
  await employees.updateMany(
    { empid: { $gte: Number(req.params.id) } },
    { $inc: { salary: Number(req.params.amount) } }
  );
  res.send("Salaries Increased");
});

app.get("/search-employee/:name", async (req, res) => {
  res.json(await employees.find({ name: req.params.name }).toArray());
});

// -------- DEPARTMENT ROUTES --------

app.get("/seed-departments", async (req, res) => {
  await departments.insertMany([
    { deptid: 101, name: "HR" },
    { deptid: 102, name: "IT" },
    { deptid: 103, name: "Finance" }
  ]);
  res.send("Departments Seeded");
});

app.get("/view-departments", async (req, res) => {
  res.json(await departments.find().toArray());
});

app.get("/add-department/:id/:name", async (req, res) => {
  await departments.insertOne({
    deptid: Number(req.params.id),
    name: req.params.name
  });
  res.send("Department Added");
});

app.get("/delete-department/:id", async (req, res) => {
  await departments.deleteOne({ deptid: Number(req.params.id) });
  res.send("Department Deleted");
});

app.get("/update-department/:id/:newname", async (req, res) => {
  await departments.updateOne(
    { deptid: Number(req.params.id) },
    { $set: { name: req.params.newname } }
  );
  res.send("Department Updated");
});

app.listen(3000);
..................................................................................................................................................................
MongoDb & Ejs - Student Management System with Pagination 
// connectDb.js
const { MongoClient } = require("mongodb");

const url = "mongodb://127.0.0.1:27017";
const dbName = "testingproject";

let db;

async function connectDb() {
  if (!db) {
    const client = await MongoClient.connect(url);
    db = client.db(dbName);
  }
  return db;
}

module.exports = connectDb;
js
Copy code
// server.js
const express = require("express");
const connectDb = require("./connectDb");

const app = express();
app.use(express.urlencoded({ extended: true }));
app.use(express.json());

app.set("view engine", "ejs");

const LIMIT = 5;

app.get("/", async (req, res) => {
  const db = await connectDb();
  const page = parseInt(req.query.page) || 1;
  const skip = (page - 1) * LIMIT;

  const filter = {};
  if (req.query.marks) filter.marks = req.query.marks;

  const students = await db
    .collection("students")
    .find(filter)
    .skip(skip)
    .limit(LIMIT)
    .toArray();

  res.render("dashboard", { students, page });
});

app.post("/add", async (req, res) => {
  const db = await connectDb();
  await db.collection("students").insertOne(req.body);
  res.redirect("/");
});

app.delete("/delete/:id", async (req, res) => {
  const db = await connectDb();
  const { ObjectId } = require("mongodb");
  await db.collection("students").deleteOne({ _id: new ObjectId(req.params.id) });
  res.sendStatus(200);
});

app.patch("/update/:id", async (req, res) => {
  const db = await connectDb();
  const { ObjectId } = require("mongodb");
  await db.collection("students").updateOne(
    { _id: new ObjectId(req.params.id) },
    { $set: req.body }
  );
  res.sendStatus(200);
});

app.listen(3000);
html
Copy code
<!-- views/dashboard.ejs -->
<!DOCTYPE html>
<html>
<head>
  <title>Student Dashboard</title>
</head>
<body>

<h2>Add Student</h2>
<form action="/add" method="post">
  <input name="name" placeholder="Name" required />
  <input name="section" placeholder="Section" required />
  <input name="marks" placeholder="Marks" required />
  <button>Add</button>
</form>

<h2>Filter by Marks</h2>
<form method="get">
  <input name="marks" placeholder="Marks" />
  <button>Filter</button>
</form>

<table border="1">
  <tr>
    <th>Name</th>
    <th>Section</th>
    <th>Marks</th>
    <th>Actions</th>
  </tr>

  <% students.forEach(s => { %>
    <tr>
      <td><%= s.name %></td>
      <td><%= s.section %></td>
      <td><%= s.marks %></td>
      <td>
        <button onclick="delStudent('<%= s._id %>')">Delete</button>
        <button onclick="updStudent('<%= s._id %>')">Update</button>
      </td>
    </tr>
  <% }) %>
</table>

<br/>
<a href="/?page=<%= page - 1 %>">Previous</a>
<a href="/?page=<%= page + 1 %>">Next</a>

<script>
function delStudent(id) {
  fetch("/delete/" + id, { method: "DELETE" })
    .then(() => location.reload());
}

function updStudent(id) {
  const name = prompt("Enter name");
  const section = prompt("Enter section");
  const marks = prompt("Enter marks");

  fetch("/update/" + id, {
    method: "PATCH",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ name, section, marks })
  }).then(() => location.reload());
}
</script>

</body>
</html>
...................................................................................................................................................................
EJS - Student Dashboard 
<!-- views/dashboard.ejs -->
<!DOCTYPE html>
<html>
<head>
  <title>Student Dashboard</title>
</head>
<body>

<h1>Welcome, <%= student.name %></h1>
<p>Email: <%= student.email %></p>

<% if (student.role === "admin") { %>
  <span>★ Admin Access</span>
<% } %>

<h2>Courses</h2>

<% if (courses.length === 0) { %>
  <p>No courses enrolled yet</p>
<% } else { %>
  <ul>
    <% courses.forEach(course => { %>
      <li>
        <%= course.title %> -
        <span style="color:
          <%= course.grade === 'A'
            ? 'green'
            : (course.grade === 'B' || course.grade === 'C')
            ? 'blue'
            : 'red' %>">
          <%= course.grade %>
        </span>
      </li>
    <% }) %>
  </ul>
<% } %>

<h3>Notice (Escaped)</h3>
<p><%= notice %></p>

<h3>Notice (Unescaped)</h3>
<p><%- notice %></p>

<%- include("footer") %>

</body>
</html>
ejs
Copy code
<!-- views/footer.ejs -->
<footer>
  <p>© 2026 Student Dashboard</p>
</footer>
.............................................................................................................................................................
EJS - Student Detail Table 
<!-- views/student-report.ejs -->
<!DOCTYPE html>
<html>
<head>
  <title>Student Report</title>
</head>
<body>

<table border="1" cellpadding="8">
  <tr>
    <th>Name</th>
    <th>Math</th>
    <th>Science</th>
    <th>English</th>
  </tr>

  <% students.forEach(student => { %>
    <tr>
      <td><%= student.name %></td>

      <td style="color:<%= student.marks.math < 70 ? 'red' : 'black' %>">
        <%= student.marks.math %>
      </td>

      <td style="color:<%= student.marks.science < 70 ? 'red' : 'black' %>">
        <%= student.marks.science %>
      </td>

      <td style="color:<%= student.marks.english < 70 ? 'red' : 'black' %>">
        <%= student.marks.english %>
      </td>
    </tr>
  <% }) %>

</table>

</body>
</html>
...................................................................................................................................................................
JWT - Token-based Authentication and Role-based Redirection User
// server.js
const express = require("express");
const fs = require("fs");
const jwt = require("jsonwebtoken");
const cookieParser = require("cookie-parser");

const app = express();
app.use(express.urlencoded({ extended: true }));
app.use(cookieParser());

app.set("view engine", "ejs");

const SECRET = "jwtsecretkey";

const users = JSON.parse(fs.readFileSync("user.json"));

app.get("/", (req, res) => {
  res.render("login", { error: null });
});

app.post("/login", (req, res) => {
  const { username, password } = req.body;

  const user = users.find(
    u => u.username === username && u.password === password
  );

  if (!user) {
    return res.render("login", { error: "Invalid Credentials" });
  }

  const token = jwt.sign(
    {
      username: user.username,
      role: user.role,
      loginTime: Date.now()
    },
    SECRET
  );

  res.cookie("token", token);

  if (user.role === "admin") return res.redirect("/adminhome");
  res.redirect("/userhome");
});

app.get("/adminhome", (req, res) => {
  res.send("Admin Home Page");
});

app.get("/userhome", (req, res) => {
  res.send("User Home Page");
});

app.listen(3000);
html
Copy code
<!-- views/login.ejs -->
<!DOCTYPE html>
<html>
<head>
  <title>Login</title>
</head>
<body>

<form method="post" action="/login">
  <input type="email" name="username" placeholder="Username" required />
  <input type="password" name="password" placeholder="Password" required />
  <button>Login</button>
</form>

<% if (error) { %>
  <p style="color:red"><%= error %></p>
<% } %>

</body>
</html>
json
Copy code
// user.json
[
  {
    "username": "user@gmail.com",
    "password": "user@123",
    "role": "user"
  },
  {
    "username": "admin@gmail.com",
    "password": "admin@123",
    "role": "admin"
  }
]
...................................................................................................................................................................
Express Session - Basic Authentication and Role-based Redirection -
// server.js
const express = require("express");
const session = require("express-session");
const fs = require("fs");

const app = express();

app.use(express.urlencoded({ extended: true }));
app.use(
  session({
    secret: "sessionsecret",
    resave: false,
    saveUninitialized: true
  })
);

app.set("view engine", "ejs");

const users = JSON.parse(fs.readFileSync("user.json"));

app.get("/", (req, res) => {
  res.render("login", { error: null });
});

app.post("/login", (req, res) => {
  const { username, password } = req.body;

  const user = users.find(
    u => u.username === username && u.password === password
  );

  if (!user) {
    return res.render("login", { error: "Invalid Credentials" });
  }

  req.session.user = {
    username: user.username,
    role: user.role,
    loginTime: Date.now()
  };

  if (user.role === "admin") return res.redirect("/adminhome");
  res.redirect("/userhome");
});

app.get("/adminhome", (req, res) => {
  if (!req.session.user || req.session.user.role !== "admin")
    return res.redirect("/");
  res.send("Admin Home Page");
});

app.get("/userhome", (req, res) => {
  if (!req.session.user || req.session.user.role !== "user")
    return res.redirect("/");
  res.send("User Home Page");
});

app.listen(3000);
html
Copy code
<!-- views/login.ejs -->
<!DOCTYPE html>
<html>
<head>
  <title>Login</title>
</head>
<body>

<form method="post" action="/login">
  <input type="email" name="username" placeholder="Username" required />
  <input type="password" name="password" placeholder="Password" required />
  <button>Login</button>
</form>

<% if (error) { %>
  <p style="color:red"><%= error %></p>
<% } %>

</body>
</html>
json
Copy code
// user.json
[
  {
    "username": "user@gmail.com",
    "password": "user@123",
    "role": "user"
  },
  {
    "username": "admin@gmail.com",
    "password": "admin@123",
    "role": "admin"
  }
]
................................................................................................................................................................
Express Session - Role-Based Concurrent Session Management with Admin Panel
// server.js
const express = require("express");
const session = require("express-session");
const fs = require("fs");

const app = express();

app.use(express.urlencoded({ extended: true }));
app.use(
  session({
    secret: "adminsessionsecret",
    resave: false,
    saveUninitialized: false,
    cookie: { maxAge: 30 * 60 * 1000 } // 30 minutes
  })
);

app.set("view engine", "ejs");

const users = JSON.parse(fs.readFileSync("user.json"));
const activeSessions = {};

app.get("/", (req, res) => {
  res.render("login", { error: null });
});

app.post("/login", (req, res) => {
  const { username, password } = req.body;

  const user = users.find(
    u => u.username === username && u.password === password
  );

  if (!user) {
    return res.render("login", { error: "Invalid Credentials" });
  }

  const loginTime = new Date();

  req.session.user = {
    username: user.username,
    role: user.role,
    loginTime
  };

  activeSessions[req.session.id] = {
    username: user.username,
    role: user.role,
    loginTime,
    expiresAt: new Date(Date.now() + req.session.cookie.maxAge)
  };

  if (user.role === "admin") return res.redirect("/adminpanel");
  res.redirect("/userhome");
});

app.get("/userhome", (req, res) => {
  if (!req.session.user || req.session.user.role !== "user")
    return res.redirect("/");
  res.send("User Home Page");
});

app.get("/adminpanel", (req, res) => {
  if (!req.session.user || req.session.user.role !== "admin")
    return res.redirect("/");

  const sessions = Object.values(activeSessions).filter(
    s => s.role === "user" || s.role === "admin"
  );

  res.render("adminpanel", { sessions });
});

app.listen(3000);
html
Copy code
<!-- views/adminpanel.ejs -->
<!DOCTYPE html>
<html>
<head>
  <title>Admin Panel</title>
</head>
<body>

<h2>Active Sessions</h2>

<table border="1" cellpadding="8">
  <tr>
    <th>Username</th>
    <th>Login Time</th>
    <th>Session Expiry</th>
  </tr>

  <% sessions.forEach(s => { %>
    <tr>
      <td><%= s.username %></td>
      <td><%= s.loginTime %></td>
      <td><%= s.expiresAt %></td>
    </tr>
  <% }) %>

</table>

</body>
</html>
html
Copy code
<!-- views/login.ejs -->
<!DOCTYPE html>
<html>
<head>
  <title>Login</title>
</head>
<body>

<form method="post" action="/login">
  <input type="email" name="username" placeholder="Username" required />
  <input type="password" name="password" placeholder="Password" required />
  <button>Login</button>
</form>

<% if (error) { %>
  <p style="color:red"><%= error %></p>
<% } %>

</body>
</html>
json
Copy code
// user.json
[
  {
    "username": "user@gmail.com",
    "password": "user@123",
    "role": "user"
  },
  {
    "username": "admin@gmail.com",
    "password": "admin@123",
    "role": "admin"
  }
]
..................................................................................................................................................................
