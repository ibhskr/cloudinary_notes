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

## cloudinaryConfig.js

```js
import { v2 as cloudinary } from "cloudinary";
import multer from "multer";
import { v4 as uuidv4 } from "uuid";
import fs from "fs";
import dotenv from "dotenv";
dotenv.config();
// process.loadEnvFile();
cloudinary.config({
  cloud_name: process.env.CLOUDINARY_CLOUD_NAME,
  api_key: process.env.CLOUDINARY_API_KEY,
  api_secret: process.env.CLOUDINARY_API_SECRET,
});

const storage = multer.diskStorage({
  destination: function (req, file, cb) {
    cb(null, "./public/temp");
  },
  filename: function (req, file, cb) {
    const uniqueName = `${uuidv4()}-${file.originalname}`;
    cb(null, uniqueName);
  },
});

export const upload = multer({ storage });
```

## router

```js
import express from "express";
const router = express.Router();
import { upload } from "../config/cloudinaryConfig.js";
import { profilePicture } from "../controllers/mediaController/profilePicture.js";
import authMiddleware from "../middlewares/authMiddleware.js";

router.post(
  "/profile-picture",
  authMiddleware,
  upload.single("myfile"),
  profilePicture
);

export default router;
```

## controller

```js
// Profile Picture Upload Handler (profilePicture.js)
import { v2 as cloudinary } from "cloudinary";
import User from "../../models/User.js";
import fs from "fs"; // You might use fs if you're cleaning up files after uploading

export const profilePicture = async (req, res) => {
  const userId = req.user.id;
  try {
    // Check if file was uploaded
    if (!req.file) {
      console.log("No file uploaded");
      return res.status(400).json({ error: "No file uploaded" });
    }

    // Upload the image to Cloudinary
    const result = await cloudinary.uploader.upload(req.file.path, {
      folder: "myFolder", // Define folder in Cloudinary
    });

    await User.findByIdAndUpdate(userId, { profilePicture: result.secure_url });

    // Return the Cloudinary URL to the client
    return res.status(200).json({
      success: true,
      message: "Profile picture uploaded successfully",
      url: result.secure_url, // Use the secure URL provided by Cloudinary
    });
  } catch (error) {
    console.error("Error uploading file:", error);
    return res.status(500).json({
      success: false,
      message: "Profile picture upload failed",
    });
  } finally {
    // Delete the file from local storage after uploading
    fs.unlink(req.file.path, (err) => {
      if (err) console.error("Error deleting file:", err);
    });
  }
};
```
