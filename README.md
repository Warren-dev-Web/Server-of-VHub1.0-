// server.js
const express = require("express");
const multer = require("multer");
const cors = require("cors");
const path = require("path");
const fs = require("fs");

const app = express();
app.use(cors());
app.use(express.json());

// Dossier uploads
const UPLOADS_DIR = path.join(__dirname, "uploads");
if (!fs.existsSync(UPLOADS_DIR)) fs.mkdirSync(UPLOADS_DIR);

// Multer config
const storage = multer.diskStorage({
  destination: (req, file, cb) => cb(null, UPLOADS_DIR),
  filename: (req, file, cb) => cb(null, Date.now() + path.extname(file.originalname))
});
const upload = multer({ storage });

// Stockage des vidéos
let videos = [];

// ----------------- ROUTES -----------------

// Upload vidéo
app.post("/upload", upload.single("video"), (req, res) => {
  const { title, uploader } = req.body;
  if (!req.file || !title || !uploader) return res.status(400).send("Manque titre, vidéo ou uploader");

  const video = {
    id: videos.length + 1,
    title,
    filename: req.file.filename,
    likes: 0,
    dislikes: 0,
    comments: [],
    uploader
  };
  videos.push(video);
  res.send({ message: "Upload OK", video });
});

// Récupérer toutes les vidéos
app.get("/videos", (req, res) => {
  res.json(videos);
});

// Like vidéo
app.post("/like", (req, res) => {
  const { id } = req.body;
  const vid = videos.find(v => v.id === id);
  if (!vid) return res.status(404).send("Vidéo non trouvée");
  vid.likes++;
  res.send({ likes: vid.likes });
});

// Dislike vidéo
app.post("/dislike", (req, res) => {
  const { id } = req.body;
  const vid = videos.find(v => v.id === id);
  if (!vid) return res.status(404).send("Vidéo non trouvée");
  vid.dislikes++;
  res.send({ dislikes: vid.dislikes });
});

// Commentaire
app.post("/comment", (req, res) => {
  const { id, user, text } = req.body;
  const vid = videos.find(v => v.id === id);
  if (!vid) return res.status(404).send("Vidéo non trouvée");
  if (!user || !text) return res.status(400).send("Utilisateur ou texte manquant");
  vid.comments.push({ user, text });
  res.send({ comments: vid.comments });
});

// Serve uploads
app.use("/uploads", express.static(UPLOADS_DIR));

// Démarrage serveur
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Serveur VHub lancé sur le port ${PORT}`));
