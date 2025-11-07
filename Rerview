// Use UMD build exposed on window to avoid module/worker issues across environments
let pdfjsLib = window.pdfjsLib;

async function ensurePdfEngine() {
  if (pdfjsLib && pdfjsLib.getDocument) {
    if (pdfjsLib.GlobalWorkerOptions && !pdfjsLib.GlobalWorkerOptions.workerSrc) {
      pdfjsLib.GlobalWorkerOptions.workerSrc = "https://cdnjs.cloudflare.com/ajax/libs/pdf.js/4.2.67/pdf.worker.min.js";
    }
    return;
  }

  await loadScript("https://cdnjs.cloudflare.com/ajax/libs/pdf.js/4.2.67/pdf.min.js");
  pdfjsLib = window.pdfjsLib;
  if (!pdfjsLib || !pdfjsLib.getDocument) {
    // try alternate CDN
    await loadScript("https://unpkg.com/pdfjs-dist@4.2.67/build/pdf.min.js");
    pdfjsLib = window.pdfjsLib;
  }
  if (!pdfjsLib || !pdfjsLib.getDocument) {
    // try jsDelivr CDN
    await loadScript("https://cdn.jsdelivr.net/npm/pdfjs-dist@4.2.67/build/pdf.min.js");
    pdfjsLib = window.pdfjsLib;
  }
  if (!pdfjsLib || !pdfjsLib.getDocument) {
    // final local fallback (user can place files under frontend/vendor)
    try {
      await loadScript("vendor/pdf.min.js");
      pdfjsLib = window.pdfjsLib;
    } catch {}
  }
  if (!pdfjsLib || !pdfjsLib.getDocument) {
    throw new Error("PDF engine could not be loaded from CDNs. See README to add local vendor files.");
  }
  if (pdfjsLib.GlobalWorkerOptions) {
    // prefer CDN worker, then local vendor fallback
    pdfjsLib.GlobalWorkerOptions.workerSrc =
      pdfjsLib.GlobalWorkerOptions.workerSrc ||
      (await pickAvailable([
        "https://cdnjs.cloudflare.com/ajax/libs/pdf.js/4.2.67/pdf.worker.min.js",
        "https://unpkg.com/pdfjs-dist@4.2.67/build/pdf.worker.min.js",
        "https://cdn.jsdelivr.net/npm/pdfjs-dist@4.2.67/build/pdf.worker.min.js",
        "vendor/pdf.worker.min.js",
      ]));
  }
}

function loadScript(src) {
  return new Promise((resolve, reject) => {
    const el = document.createElement("script");
    el.src = src;
    el.crossOrigin = "anonymous";
    el.onload = () => resolve();
    el.onerror = () => reject(new Error(`Failed to load script: ${src}`));
    document.head.appendChild(el);
  });
}

async function pickAvailable(urls) {
  for (const url of urls) {
    try {
      await fetch(url, { method: "HEAD", mode: "no-cors" });
      return url; // best-effort; HEAD may be opaque in no-cors but proceed
    } catch {}
  }
  return urls[urls.length - 1];
}

const TECHNICAL_SKILLS = [
  "python",
  "java",
  "c++",
  "c#",
  "javascript",
  "typescript",
  "react",
  "node",
  "node.js",
  "terraform",
  "figma",
  "adobe xd",
  "express",
  "django",
  "flask",
  "sql",
  "mysql",
  "postgres",
  "mongodb",
  "aws",
  "azure",
  "gcp",
  "docker",
  "kubernetes",
  "git",
  "html",
  "css",
  "tailwind",
  "tensorflow",
  "pytorch",
  "machine learning",
  "data analysis",
  "power bi",
  "tableau",
  "excel",
  "bash",
  "linux",
  "rest",
  "graphql",
  "nlp",
  "rust",
  "go",
];

const SOFT_SKILLS = [
  "communication",
  "leadership",
  "collaboration",
  "problem solving",
  "critical thinking",
  "teamwork",
  "adaptability",
  "time management",
  "mentoring",
  "creativity",
  "presentation",
  "stakeholder management",
  "strategic planning",
  "customer focus",
];

const JOB_PROFILES = [
  {
    title: "Software Engineer",
    technical: ["python", "java", "c++", "c#", "javascript", "react", "node", "node.js"],
    soft: ["problem solving", "teamwork", "communication"],
  },
  {
    title: "Frontend Developer",
    technical: ["javascript", "typescript", "react", "html", "css", "tailwind"],
    soft: ["creativity", "communication"],
  },
  {
    title: "Backend Developer",
    technical: ["python", "java", "node", "node.js", "express", "sql", "docker", "aws"],
    soft: ["problem solving", "teamwork"],
  },
  {
    title: "Data Analyst",
    technical: ["python", "sql", "excel", "tableau", "power bi", "data analysis"],
    soft: ["critical thinking", "communication"],
  },
  {
    title: "Data Scientist",
    technical: ["python", "machine learning", "pytorch", "tensorflow", "sql", "nlp"],
    soft: ["problem solving", "communication"],
  },
  {
    title: "DevOps Engineer",
    technical: ["aws", "azure", "gcp", "docker", "kubernetes", "linux", "bash"],
    soft: ["communication", "stakeholder management"],
  },
  {
    title: "Product Manager",
    technical: ["sql", "data analysis", "excel"],
    soft: ["leadership", "strategic planning", "communication"],
  },
  {
    title: "UI/UX Designer",
    technical: ["figma", "adobe xd", "html", "css"],
    soft: ["creativity", "communication", "customer focus"],
  },
  {
    title: "Cloud Engineer",
    technical: ["aws", "azure", "gcp", "terraform", "docker", "kubernetes"],
    soft: ["problem solving", "communication"],
  },
  {
    title: "Business Analyst",
    technical: ["sql", "excel", "power bi", "tableau"],
    soft: ["stakeholder management", "communication", "critical thinking"],
  },
];

const form = document.getElementById("resume-form");
const fileInput = document.getElementById("resume-file");
const textInput = document.getElementById("resume-text");
const dropzone = document.getElementById("dropzone");
const statusBox = document.getElementById("status");
const analyzeButton = document.getElementById("analyze-btn");
const forceOcrCheckbox = document.getElementById("force-ocr");
const fileChip = document.getElementById("file-chip");
const fileNameLabel = document.getElementById("file-name");
const clearFileButton = document.getElementById("clear-file");
const resultsSection = document.getElementById("results");
const jobList = document.getElementById("job-list");
const technicalList = document.getElementById("technical-skills");
const softList = document.getElementById("soft-skills");
const techCount = document.getElementById("tech-count");
const softCount = document.getElementById("soft-count");
const roleCount = document.getElementById("role-count");
const resultSubtitle = document.getElementById("result-subtitle");

const MAX_FILE_SIZE_MB = 5;

form.addEventListener("submit", async (event) => {
  event.preventDefault();
  resetResults();
  setStatus("Analyzing resume…", "info");
  setLoading(true);

  try {
    const analysis = await analyseResume();

    if (!analysis.recommendations.length) {
      setStatus(
        "We could not find enough skills to suggest specific roles. Try adding more detail to your resume.",
        "warning"
      );
      return;
    }

    renderResults(analysis);
    setStatus("Analysis complete. Explore the recommendations below.", "success");
  } catch (error) {
    console.error(error);
    setStatus(error.message || "Unable to analyze resume.", "error");
  } finally {
    setLoading(false);
  }
});

fileInput.addEventListener("change", () => {
  updateFileChip();
  setStatus("Resume loaded. Add optional text or analyze now.", "info");
});

clearFileButton.addEventListener("click", () => {
  fileInput.value = "";
  updateFileChip();
});

setupDropzone();
setStatus("Upload a resume to get tailored job suggestions.", "info");

async function analyseResume() {
  const file = fileInput.files[0];
  const pastedText = textInput.value.trim();

  if (!file && !pastedText) {
    throw new Error("Upload a resume file or paste your resume text to continue.");
  }

  if (file && file.size / (1024 * 1024) > MAX_FILE_SIZE_MB) {
    throw new Error(`Please choose a file smaller than ${MAX_FILE_SIZE_MB} MB.`);
  }

  let resumeText = pastedText;
  if (file) {
    resumeText = await readFileContents(file, { forceOcr: !!forceOcrCheckbox?.checked });
  }

  if (!resumeText || !resumeText.trim()) {
    throw new Error("We could not read any text from the provided resume.");
  }

  const { technical, soft } = extractSkills(resumeText);
  const recommendations = suggestJobTitles(technical, soft, 5);

  return {
    resumeText,
    technicalSkills: [...technical].sort(),
    softSkills: [...soft].sort(),
    recommendations,
  };
}

async function readFileContents(file, { forceOcr = false } = {}) {
  if (file.type === "application/pdf" || file.name.toLowerCase().endsWith(".pdf")) {
    await ensurePdfEngine();
    return readPdf(file, { forceOcr });
  }

  return file.text();
}

async function readPdf(file, { forceOcr = false } = {}) {
  const data = await file.arrayBuffer();
  await ensurePdfEngine();
  const pdf = await pdfjsLib.getDocument({ data }).promise;
  let text = "";

  for (let pageNumber = 1; pageNumber <= pdf.numPages; pageNumber += 1) {
    const page = await pdf.getPage(pageNumber);
    try {
      const content = await page.getTextContent({ normalizeWhitespace: true });
      const strings = content.items.map((item) => item.str).filter(Boolean);
      text += `${strings.join(" ")}\n`;
    } catch (e) {
      console.warn("Text extraction failed on page", pageNumber, e);
    }
  }

  const trimmed = text.trim();
  if (!forceOcr && (trimmed.length >= 120 || !window.Tesseract)) {
    return trimmed;
  }

  // Fallback to OCR for scanned/image-only PDFs (first few pages for speed)
  setStatus(forceOcr ? "Running OCR as requested…" : "No selectable text found. Running OCR on first pages…", "info");
  try {
    const ocrText = await ocrFirstPages(pdf, 3);
    return ocrText.trim() || trimmed;
  } catch (err) {
    console.warn("OCR failed:", err);
    return trimmed;
  }
}

async function ocrFirstPages(pdf, maxPages = 2) {
  let aggregated = "";
  const pages = Math.min(maxPages, pdf.numPages);
  for (let i = 1; i <= pages; i += 1) {
    setStatus(`OCR: processing page ${i}/${pages}…`, "info");
    const page = await pdf.getPage(i);
    const canvas = await renderPageToCanvas(page, 1.75);
    // Tesseract.js global from CDN
    const { data } = await window.Tesseract.recognize(canvas, "eng", {
      logger: (m) => {
        if (m?.status === "recognizing text" && m?.progress != null) {
          const pct = Math.round(m.progress * 100);
          setStatus(`OCR: page ${i}/${pages} – ${pct}%`, "info");
        }
      },
    });
    aggregated += `\n${data?.text || ""}`;
  }
  return aggregated;
}

async function renderPageToCanvas(page, scale = 1.5) {
  const viewport = page.getViewport({ scale });
  const canvas = document.createElement("canvas");
  const context = canvas.getContext("2d");
  canvas.width = viewport.width;
  canvas.height = viewport.height;
  await page.render({ canvasContext: context, viewport }).promise;
  return canvas;
}

function extractSkills(resumeText) {
  const normalized = normalizeText(resumeText);
  const technicalMatches = new Set();
  const softMatches = new Set();

  TECHNICAL_SKILLS.forEach((skill) => {
    if (containsSkill(normalized, skill)) {
      technicalMatches.add(skill);
    }
  });

  SOFT_SKILLS.forEach((skill) => {
    if (containsSkill(normalized, skill)) {
      softMatches.add(skill);
    }
  });

  return { technical: technicalMatches, soft: softMatches };
}

function containsSkill(text, rawSkill) {
  const skill = rawSkill.toLowerCase();
  if (text.includes(skill)) {
    return true;
  }

  const escaped = escapeRegExp(skill);
  const pattern = new RegExp(`(^|[^\\w])${escaped}([^\\w]|$)`, "i");
  return pattern.test(text);
}

function normalizeText(text) {
  return text.toLowerCase().replace(/\s+/g, " ").trim();
}

function escapeRegExp(value) {
  return value.replace(/[.*+?^${}()|[\]\\]/g, "\\$&");
}

function suggestJobTitles(technicalSkills, softSkills, topN = 5) {
  const techSet = new Set(technicalSkills);
  const softSet = new Set(softSkills);

  const scored = JOB_PROFILES.map((profile) => {
    const techOverlap = profile.technical.filter((skill) => techSet.has(skill));
    const softOverlap = profile.soft.filter((skill) => softSet.has(skill));
    const score = techOverlap.length * 2 + softOverlap.length;

    return {
      title: profile.title,
      score,
      reasons: [
        techOverlap.length ? `Technical: ${techOverlap.join(", ")}` : "",
        softOverlap.length ? `Soft: ${softOverlap.join(", ")}` : "",
      ].filter(Boolean),
    };
  })
    .filter((item) => item.score > 0)
    .sort((a, b) => b.score - a.score);

  return scored.slice(0, topN);
}

function renderResults({ technicalSkills, softSkills, recommendations }) {
  jobList.innerHTML = "";
  technicalList.innerHTML = "";
  softList.innerHTML = "";

  techCount.textContent = technicalSkills.length;
  softCount.textContent = softSkills.length;
  roleCount.textContent = recommendations.length;
  resultSubtitle.textContent = `Matched using ${technicalSkills.length + softSkills.length} verified skills from your resume.`;

  recommendations.forEach((item) => {
    const card = document.createElement("article");
    card.className = "job-card";

    const header = document.createElement("div");
    header.className = "job-card__header";

    const title = document.createElement("h3");
    title.className = "job-card__title";
    title.textContent = item.title;

    const score = document.createElement("span");
    score.className = "job-card__score";
    score.textContent = `Score ${item.score}`;

    header.append(title, score);
    card.append(header);

    if (item.reasons.length) {
      const list = document.createElement("ul");
      list.className = "job-card__reasons";
      item.reasons.forEach((reason) => {
        const li = document.createElement("li");
        li.textContent = reason;
        list.appendChild(li);
      });
      card.append(list);
    }

    jobList.append(card);
  });

  updateSkillList(technicalList, technicalSkills);
  updateSkillList(softList, softSkills);

  resultsSection.hidden = false;
}

function updateSkillList(target, skills) {
  if (!skills.length) {
    const placeholder = document.createElement("li");
    placeholder.textContent = "No matches yet. Add more detail to this area of your resume.";
    target.appendChild(placeholder);
    return;
  }

  skills.forEach((skill) => {
    const li = document.createElement("li");
    li.textContent = skill;
    target.appendChild(li);
  });
}

function resetResults() {
  resultsSection.hidden = true;
  jobList.innerHTML = "";
  technicalList.innerHTML = "";
  softList.innerHTML = "";
  techCount.textContent = "0";
  softCount.textContent = "0";
  roleCount.textContent = "0";
}

function setStatus(message, type = "info") {
  statusBox.textContent = message;
  statusBox.dataset.type = type;
}

function setLoading(isLoading) {
  analyzeButton.disabled = isLoading;
  analyzeButton.classList.toggle("is-loading", isLoading);
}

function updateFileChip() {
  const file = fileInput.files[0];
  if (file) {
    fileNameLabel.textContent = `${file.name} · ${(file.size / (1024 * 1024)).toFixed(2)} MB`;
    fileChip.hidden = false;
  } else {
    fileChip.hidden = true;
    fileNameLabel.textContent = "";
  }
}

function setupDropzone() {
  ["dragenter", "dragover"].forEach((eventName) => {
    dropzone.addEventListener(eventName, (event) => {
      event.preventDefault();
      event.stopPropagation();
      dropzone.classList.add("is-dragover");
    });
  });

  ["dragleave", "drop"].forEach((eventName) => {
    dropzone.addEventListener(eventName, (event) => {
      event.preventDefault();
      event.stopPropagation();
      dropzone.classList.remove("is-dragover");
    });
  });

  dropzone.addEventListener("drop", (event) => {
    const files = event.dataTransfer.files;
    if (!files?.length) {
      return;
    }

    fileInput.files = files;
    updateFileChip();
    setStatus("Resume loaded. Add optional text or analyze now.", "info");
  });
}

