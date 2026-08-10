# Scaling Guacamole

Website: [www.scaling-guacamole.com](https://www.scaling-guacamole.com/)

An accessible static blog featuring developer productivity, AI-assisted
development, and software engineering practices.

## Features

- **Responsive Design**: Works beautifully on desktop, tablet, and mobile devices
- **Dark/Light Mode**: Automatic theme detection with manual toggle (Alt + T)
- **Gradient Design**: Beautiful purple gradient theme throughout
- **Accessible**: WCAG compliant with keyboard navigation and screen reader support
- **Static Site**: Pure HTML, CSS, and JavaScript with no build step
- **Automated Deployment**: Published through GitHub Pages and Azure Static Web
   Apps

## Local Development

1. Clone the repository:

   ```bash
   git clone https://github.com/codess-aus/Scaling-Guacamole.git
   cd Scaling-Guacamole
   ```

2. Open `index.html` in your browser or use a local server:

   ```bash
   python -m http.server 8000
   # or
   npx serve
   ```

3. Visit `http://localhost:8000` in your browser

## Publishing Blog Posts

Blog source material can be drafted in `docs/`, but only HTML files in `posts/`
are published as articles. Publishing also requires a matching card in
`index.html` and any referenced assets in `images/`.

See [docs/PUBLISHING.md](docs/PUBLISHING.md) for the complete authoring,
conversion, validation, and deployment workflow.

## Deployment

Pushes to `main` deploy the static site through both configured workflows:

- `.github/workflows/deploy.yml` publishes to GitHub Pages.
- `.github/workflows/azure-static-web-apps-green-stone-03a17880f.yml` publishes
   to Azure Static Web Apps.

Pull requests targeting `main` also create or update an Azure Static Web Apps
preview environment. Closing the pull request removes that preview.

### First-time Setup

1. Go to your repository settings on GitHub
2. Navigate to **Pages** in the sidebar
3. Under **Source**, select "GitHub Actions"
4. The workflow will deploy on every push to `main`.

### Manual Deployment

You can also trigger deployment manually:

1. Go to the **Actions** tab in your repository
2. Select the "Deploy to GitHub Pages" workflow
3. Click "Run workflow"

## Customization

### Colors

Edit the CSS variables in `styles.css`:

```css
:root {
    --accent-from: #667eea;  /* Gradient start color */
    --accent-to: #764ba2;    /* Gradient end color */
}
```

### Theme

The theme persists using localStorage and respects system preferences by default.

## Accessibility Features

- Semantic HTML structure
- ARIA labels and roles
- Keyboard navigation support
- Skip to main content link
- Focus indicators
- Reduced motion support
- High contrast in both themes

## Browser Support

- Chrome/Edge (latest)
- Firefox (latest)
- Safari (latest)
- Mobile browsers (iOS Safari, Chrome Mobile)

## License

See [LICENSE](LICENSE) file for details.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

When contributing a post, follow the publishing checklist and include the
source Markdown when one exists.
