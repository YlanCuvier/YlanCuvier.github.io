# Vue Inspira UI

A modern Vue.js project with beautiful Inspira UI components built with Tailwind CSS.

## Features

- Vue 3 with Composition API
- Tailwind CSS for styling
- Modern UI components
- Responsive design
- TypeScript support ready

## Getting Started

### Prerequisites

- Node.js (version 16 or higher)
- npm or yarn

### Installation

1. Install dependencies:
```bash
npm install
```

2. Start the development server:
```bash
npm run dev
```

3. Open your browser and navigate to `http://localhost:3000`

### Available Scripts

- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run preview` - Preview production build

## Deploy to GitHub Pages

This project is configured for GitHub Pages with:

- Relative asset paths in `vite.config.js` (`base: './'`)
- An automated workflow in `.github/workflows/deploy.yml`

To activate deployment:

1. Push this repository to GitHub (branch `main`).
2. In GitHub, go to **Settings > Pages**.
3. Under **Build and deployment**, choose **Source: GitHub Actions**.
4. Push changes to `main` and the site will deploy automatically.

## Components

The project includes the following Inspira UI components:

- **InspiraButton** - Various button styles and states
- **InspiraCard** - Card container component
- **InspiraInput** - Form input fields
- **InspiraAlert** - Notification and alert components
- **InspiraProgress** - Progress indicators
- **InspiraModal** - Dialog and modal components
- **InspiraBadge** - Status indicators and labels

## Project Structure

```
src/
├── components/          # Inspira UI components
├── App.vue             # Main application component
├── main.js             # Application entry point
└── style.css           # Global styles and Tailwind imports
```

## Customization

You can customize the design system by modifying:

- `tailwind.config.js` - Tailwind CSS configuration
- `src/style.css` - Global styles and component classes
- Individual component files in `src/components/`

## License

MIT
