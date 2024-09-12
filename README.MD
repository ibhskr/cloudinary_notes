### File Upload API with Express, Multer, and Cloudinary
## Main Features:
- Secure file uploading using **Multer**.
- generate uniqe name using **uuid**.
- Stores files in **Cloudinary**.
- Removes uploaded files from local storage after successful upload.
- then we need to store "secure_url" to our database;


## Install Dependencies

```bash
npm i multer
npm i uuid
npm i cloudinary
```



### File Structure

```plaintext
├── db/
│   └── db.js               # Database configuration (optional)
├── uploads/                # Directory for temporarily storing uploaded files
├── .env                    # Environment variables file
├── index.js                # Main application file
├── package.json            # Dependencies and scripts
└── README.md               # Project documentation
```

## index.js

```js
import express from "express";
import multer from "multer";
import { v4 as uuidv4 } from "uuid";
import { v2 as cloudinary } from "cloudinary";
import fs from "fs";
import path from "path";

process.loadEnvFile(); // Load environment variables from .env file
import "./db/db.js"; // Import database configuration

const app = express();

// Cloudinary configuration using environment variables
cloudinary.config({
  cloud_name: process.env.CLOUDINARY_CLOUD_NAME,
  api_key: process.env.CLOUDINARY_API_KEY,
  api_secret: process.env.CLOUDINARY_API_SECRET,
});

// Multer configuration for file storage
const storage = multer.diskStorage({
  destination: function (req, file, cb) {
    cb(null, "./uploads");
  },
  filename: function (req, file, cb) {
    const uniqueName = `${uuidv4()}-${file.originalname}`;
    cb(null, uniqueName);
  },
});

const upload = multer({ storage });

// Route to handle file uploads

app.post("/", upload.single("myfile"), async (req, res) => {
  try {
    if (!req.file) {
      return res.status(400).json({ error: "No file uploaded" });
    }

    // Upload file to Cloudinary
    const uploadResult = await cloudinary.uploader.upload(req.file.path);

    // Remove file from local storage after upload
    fs.unlink(req.file.path, (err) => {
      if (err) {
        console.error("Error deleting file:", err);
      } else {
        console.log("File successfully deleted from local storage");
      }
    });
    console.log(uploadResult.secure_url);
    res.status(200).json(uploadResult);
  } catch (error) {
    console.error("Error uploading file:", error);
    res.status(500).json({ error: "File upload failed" });
  }
});

// Start the server

const PORT = process.env.PORT || 8000;
app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});

```