# Daily Developer Inspiration

A simple, elegant web application that delivers daily programming wisdom to inspire and educate developers. Each weekday, discover a new piece of expert advice to enhance your coding journey.

## 🎯 About

This site provides curated daily programming insights sourced from **"97 Things Every Programmer Should Know"** - a collection of wisdom from experienced software developers and industry experts. Each article offers practical advice, best practices, and thoughtful perspectives on software development.

**Credit**: All articles are sourced from *"97 Things Every Programmer Should Know"* and full credit goes to the original authors and contributors of that excellent resource.

## ✨ Features

- **📅 Daily Articles**: New programming wisdom delivered every weekday (Monday-Friday)
- **🌙 Dark Mode**: Beautiful, easy-on-the-eyes reading experience
- **📱 Responsive Design**: Optimized for desktop, tablet, and mobile devices
- **⚡ Fast & Lightweight**: Built with Vue.js and Vite for optimal performance
- **🧭 Smart Navigation**: Previous/Next navigation with intelligent unlock messaging
- **📖 Markdown Rendering**: Clean, formatted content for better readability
- **🔒 Scheduled Content**: Articles unlock only on weekdays, respecting the daily rhythm

## 🚀 Live Demo

Visit the live application: [Daily Developer Inspiration](https://james-ws-mccullough.github.io/daily-developer-inspiration-webapp/)

## 🛠 Technology Stack

- **Frontend**: Vue.js 3 with Composition API
- **Build Tool**: Vite
- **Styling**: CSS with modern features (backdrop-filter, gradients, etc.)
- **Markdown**: Marked.js for content rendering
- **Deployment**: GitHub Pages with automated CI/CD

## 📦 Installation & Development

### Prerequisites
- Node.js 20.19+ or 22.12+
- npm or yarn

### Getting Started

1. **Clone the repository**
   ```bash
   git clone https://github.com/James-WS-McCullough/daily-developer-inspiration-webapp.git
   cd daily-developer-inspiration-webapp
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Start development server**
   ```bash
   npm run dev
   ```

4. **Build for production**
   ```bash
   npm run build
   ```

## 🎮 Development Features

When running in development mode (`npm run dev`), additional features are available:

- **Developer Navigation**: Access to all articles, including future ones
- **Article Counter**: See "Day X of Y" indicator in the header
- **Full Timeline**: Navigate freely through the entire article collection

## 📁 Project Structure

```
src/
├── components/
│   └── DailyInspiration.vue    # Main application component
├── data/
│   └── articles.json           # Article content and metadata
├── style.css                   # Global styles
├── App.vue                     # Root component
└── main.js                     # Application entry point
```

### Adding Articles
Add new articles to `src/data/articles.json` following this structure:

```json
{
  "inspiration": [
    {
      "title": "Article Title",
      "author": "Author Name",
      "content": "Article content in markdown format..."
    }
  ]
}
```

## 🚀 Deployment

The project includes automated GitHub Pages deployment via GitHub Actions. Simply push to the `main` branch and the site will be automatically built and deployed.

Manual deployment is also available:
```bash
npm run build && npm run deploy
```

## 📚 Content Attribution

All articles are sourced from **"97 Things Every Programmer Should Know"**, a collaborative book featuring insights from experienced programmers and software development experts. We gratefully acknowledge and credit the original authors and contributors for their valuable wisdom.

---

*Daily wisdom for the developer's journey* ✨
