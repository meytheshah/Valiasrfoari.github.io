<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>News</title>
<style>
<center>اخبار صحن ولیعصر</center>

body {
  background: #0f172a;
  color: #e5e7eb;
  font-family: sans-serif;
  margin: 0;
  padding: 20px;
}
.article {
  max-width: 800px;
  margin: 40px auto;
  background: #1e293b; /* Slightly lighter than body for contrast */
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.5);
}
.article img {
  width: 100%;
  height: auto;
  border-radius: 10px;
  object-fit: cover;
  max-height: 400px;
}
h2 {
  margin-top: 15px;
  color: #38bdf8;
}
p {
  line-height: 1.6;
}
.error {
  color: #ef4444;
  text-align: center;
}
</style>
</head>
<body>

<div id="news">Loading news...</div>

<script>
const newsContainer = document.getElementById("news");

// Helper function to fetch text safely
async function fetchText(url) {
  const response = await fetch(url);
  if (!response.ok) throw new Error(`Failed to load ${url}`);
  return await response.text();
}

// Main execution
async function loadNews() {
  try {
    // 1. Fetch the index list
    const response = await fetch("articles/index.json");
    if (!response.ok) throw new Error("Failed to load article list");
    
    const list = await response.json();
    
    // Clear loading text
    newsContainer.innerHTML = "";

    // 2. Process each article
    // We use Promise.all to ensure all articles load, or map to handle them individually
    const articlePromises = list.map(async (folder) => {
      try {
        // Fetch text first so we have all data before rendering
        const textContent = await fetchText(`articles/${folder}/text.txt`);
        
        const container = document.createElement("div");
        container.className = "article";

        const title = document.createElement("h2");
        // Assuming folder name is the title, or you could parse it
        title.textContent = folder.replace(/-/g, " ").replace(/\b\w/g, l => l.toUpperCase()); 

        const img = document.createElement("img");
        img.src = `articles/${folder}/image.jpg`;
        img.alt = folder; // Good for accessibility
        img.onerror = () => { img.style.display = 'none'; }; // Hide img if broken

        const text = document.createElement("div");
        text.className = "content";
        text.innerText = textContent; // Use innerText for safety against XSS

        container.append(img, title, text);
        return container;
      } catch (err) {
        console.error(`Error loading article ${folder}:`, err);
        return null; // Skip this article if it fails
      }
    });

    // Wait for all articles to be prepared
    const articles = await Promise.all(articlePromises);

    // Append valid articles to the DOM
    articles.forEach(article => {
      if (article) newsContainer.append(article);
    });

    if (newsContainer.children.length === 0) {
      newsContainer.innerHTML = "<p class='error'>No articles found.</p>";
    }

  } catch (error) {
    console.error(error);
    newsContainer.innerHTML = `<p class='error'>Error loading news feed: ${error.message}</p>`;
  }
}

loadNews();
</script>

</body>
</html>
