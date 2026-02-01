# Google-Drive-View-Only-PDF-Downloader

Here you can use this script to download **view-only PDF files from Google Drive**.

This script works by capturing all PDF pages rendered in the browser as high-resolution images and then combining them into a single downloadable PDF file using `jsPDF`.

> ⚠️ **Big Note:**  
> Use this script wisely and ethically.  
> It is intended for personal notes, educational material, or files you are allowed to view.  
> Do NOT misuse it for copyrighted or restricted content.

---

## 📌 How This Script Works (Concept)

Google Drive does **not** stream PDFs directly when a file is set to *view-only*.  
Instead, it:
- Renders **each page as a blob image**
- Displays them lazily as you scroll

This script:
1. Detects those blob-based page images
2. Converts each page into a canvas
3. Rebuilds all pages into a new PDF
4. Triggers a browser download

You are **reconstructing the PDF**, not downloading it from Drive.

---

## 🛠️ Instructions (Important – Follow Carefully)

1. Open the PDF from **Google Drive**
2. Click **Preview**
3. On the top-right corner, click **three vertical dots → Open in new window**
4. **Scroll down to the very end** of the PDF  
   (This ensures all pages are fully loaded)
5. Open **Inspect Element / DevTools**
6. Go to the **Console** tab
7. If Chrome blocks pasting: allow pasting
8. Paste the script and press Enter(execute).

---


## 🧠 Key insight:
If something is visible pixel-by-pixel in the browser, JavaScript can access it.

## What is a Blob?
A Blob is a browser-managed object that temporarily holds raw binary data (images, PDFs, videos) in memory without exposing it as a direct downloadable file.

---
---


```javascript
(function () {
  // Log to confirm script execution
  console.log("Loading script ...");

  // Dynamically create a <script> tag to load jsPDF
  let script = document.createElement("script");

  // Run this function once jsPDF is fully loaded
  script.onload = function () {

    // Extract jsPDF constructor from the UMD bundle
    const { jsPDF } = window.jspdf;

    // PDF object (initialized later)
    let pdf = null;

    // Collect all <img> elements on the page
    let imgElements = document.getElementsByTagName("img");

    // Store only valid Google Drive PDF page images
    let validImgs = [];

    console.log("Scanning content ...");

    // Loop through every image on the page
    for (let i = 0; i < imgElements.length; i++) {
      let img = imgElements[i];

      // Google Drive renders PDF pages as blob images
      let checkURLString = "blob:https://drive.google.com/";

      // Ignore non-PDF images
      if (img.src.substring(0, checkURLString.length) !== checkURLString) {
        continue;
      }

      // Store valid PDF page image
      validImgs.push(img);
    }

    console.log(`${validImgs.length} content found!`);
    console.log("Generating PDF file ...");

    // Process each page image
    for (let i = 0; i < validImgs.length; i++) {
      let img = validImgs[i];

      // Create a canvas to convert image → base64
      let canvasElement = document.createElement("canvas");
      let con = canvasElement.getContext("2d");

      // Match original resolution to avoid blur
      canvasElement.width = img.naturalWidth;
      canvasElement.height = img.naturalHeight;

      // Draw image on canvas
      con.drawImage(img, 0, 0, img.naturalWidth, img.naturalHeight);

      // Convert canvas image to base64 PNG
      let imgData = canvasElement.toDataURL();

      // Determine orientation per page
      let orientation = img.naturalWidth > img.naturalHeight ? "l" : "p";

      // Page dimensions
      let pageWidth = img.naturalWidth;
      let pageHeight = img.naturalHeight;

      // Initialize PDF on first page
      if (i === 0) {
        pdf = new jsPDF({
          orientation: orientation,
          unit: "px",
          format: [pageWidth, pageHeight],
        });
      } else {
        // Add new page for subsequent images
        pdf.addPage([pageWidth, pageHeight], orientation);
      }

      // Insert image into PDF page
      pdf.addImage(
        imgData,
        "PNG",
        0,
        0,
        pageWidth,
        pageHeight,
        "",
        "SLOW" // higher quality rendering
      );

      // Progress feedback
      const percentages = Math.floor(((i + 1) / validImgs.length) * 100);
      console.log(`Processing content ${percentages}%`);
    }

    // Extract file name from page metadata
    let title =
      document.querySelector('meta[itemprop="name"]')?.content ||
      document.title ||
      "download.pdf";

    // Ensure .pdf extension
    if ((title.split(".").pop() || "").toLowerCase() !== "pdf") {
      title = title + ".pdf";
    }

    console.log("Downloading PDF file ...");

    // Trigger browser download
    pdf.save(title, { returnPromise: true }).then(() => {
      document.body.removeChild(script);
      console.log("PDF downloaded!");
    });
  };

  // Load jsPDF from a trusted CDN
  let scriptURL = "https://unpkg.com/jspdf@latest/dist/jspdf.umd.min.js";
  let trustedURL;

  // Handle Chrome Trusted Types security
  if (window.trustedTypes && trustedTypes.createPolicy) {
    const policy = trustedTypes.createPolicy("myPolicy", {
      createScriptURL: (input) => input,
    });
    trustedURL = policy.createScriptURL(scriptURL);
  } else {
    trustedURL = scriptURL;
  }

  // Attach jsPDF script to document
  script.src = trustedURL;
  document.body.appendChild(script);
})();
```

---

## How This Bypasses “View-Only” Restriction

- Google Drive already sends page data to your browse
- Pages are rendered as blob images

- JavaScript has full access to rendered DOM content

- Canvas allows pixel-level extraction

- jsPDF reconstructs the document
---
---

> ⚠️ If content is visible in the browser, - it can be programmatically captured.