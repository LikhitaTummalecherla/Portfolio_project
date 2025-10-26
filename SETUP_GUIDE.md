# Portfolio Website Setup Guide

## Prerequisites Check

Before starting, verify you have these installed:

```bash
# Check Node.js version (should be 18+)
node --version

# Check npm version
npm --version

# Check Docker version (optional, for Docker setup)
docker --version

# Check Docker Compose version (optional)
docker-compose --version
```

If any of these commands fail, install the missing software first.

## Quick Setup (Recommended)

### Step 1: Navigate to Project Directory
```bash
# Replace 'your-project-folder' with the actual folder name
cd your-project-folder

# Or if you're in a parent directory:
cd path/to/your-project-folder
```

### Step 2: Install Dependencies
```bash
npm install
```

### Step 3: Start Development Server
```bash
npm run dev
```

**That's it!** Your portfolio will be available at: http://localhost:3000

## Alternative Setup Methods

### Method 1: Docker Development (Recommended for Production-like Environment)
```bash
# Build and start development container
docker-compose up --build portfolio-dev

# Or run in background
docker-compose up -d --build portfolio-dev

# View logs
docker-compose logs -f portfolio-dev

# Stop when done
docker-compose down
```

### Method 2: Docker Production Build
```bash
# Build and start production container
docker-compose up --build portfolio-prod

# Access at: http://localhost:8080
```

## Available Commands

### Development Commands
```bash
npm run dev          # Start development server (hot reload)
npm run build        # Build for production
npm run preview      # Preview production build locally
npm test             # Run tests
npm run test:watch   # Run tests in watch mode
npm run lint         # Check code quality
npm run lint:fix     # Fix linting issues automatically
```

### Docker Commands
```bash
# Development
docker-compose up portfolio-dev              # Start dev server
docker-compose up --build portfolio-dev      # Rebuild and start
docker-compose logs portfolio-dev            # View logs
docker-compose logs -f portfolio-dev         # Follow logs live

# Production
docker-compose up portfolio-prod             # Start production server
docker-compose up --build portfolio-prod     # Rebuild and start production

# Management
docker-compose down                          # Stop all services
docker-compose stop portfolio-dev            # Stop specific service
docker-compose restart portfolio-dev         # Restart service
```

## Troubleshooting

### Common Issues and Solutions

#### 1. "Command not found" errors
```bash
# If 'npm' command not found, install Node.js from:
# https://nodejs.org/

# If 'docker' command not found, install Docker from:
# https://www.docker.com/get-started
```

#### 2. Port already in use (EADDRINUSE)
```bash
# Find and kill process using port 3000
# Windows:
netstat -ano | findstr :3000
taskkill /PID <PID_NUMBER> /F

# Mac/Linux:
lsof -ti:3000 | xargs kill -9

# Or use different port:
npm run dev -- --port 3001
```

#### 3. Permission denied (Docker on Linux/Mac)
```bash
# Add your user to docker group
sudo usermod -aG docker $USER

# Then logout and login again, or run:
newgrp docker
```

#### 4. Node modules issues
```bash
# Clear cache and reinstall
rm -rf node_modules package-lock.json
npm install

# Or on Windows:
rmdir /s node_modules
del package-lock.json
npm install
```

#### 5. Docker build fails
```bash
# Clean Docker cache
docker system prune -a
docker-compose build --no-cache portfolio-dev
```

## Project Structure

```
portfolio-website/
├── src/
│   ├── main.js          # Main JavaScript functionality
│   ├── styles.css       # All CSS styles
│   └── *.test.js        # Test files
├── public/
│   ├── favicon.svg      # Website icon
│   └── sw.js           # Service worker for PWA
├── k8s/                # Kubernetes deployment files
├── docker-compose.yml  # Docker services configuration
├── Dockerfile          # Multi-stage Docker build
├── Jenkinsfile         # CI/CD pipeline configuration
├── vite.config.js      # Vite build configuration
├── package.json        # Dependencies and scripts
└── index.html          # Main HTML file
```

## Customization Guide

### 1. Update Personal Information
Edit `index.html` and update:
- Your name in the hero section
- About section content
- Skills and technologies
- Project information
- Contact details and social links

### 2. Modify Styling
Edit `src/styles.css` to customize:
- Color scheme (search for `#667eea` and `#764ba2`)
- Fonts and typography
- Layout and spacing
- Animations and effects

### 3. Add New Features
Edit `src/main.js` to add:
- New interactive elements
- Additional animations
- Custom form handling
- Analytics tracking

## Deployment Options

### Local Development
```bash
npm run dev
# Perfect for development with hot reloading
# Access: http://localhost:3000
```

### Local Production Preview
```bash
npm run build
npm run preview
# Test production build locally
# Access: http://localhost:4173
```

### Docker Development
```bash
docker-compose up portfolio-dev
# Development with Docker
# Access: http://localhost:3000
```

### Docker Production
```bash
docker-compose up portfolio-prod
# Production build with nginx
# Access: http://localhost:8080
```

## Next Steps

1. **Start with local development**: `npm run dev`
2. **Customize content** to match your profile
3. **Test responsive design** on different screen sizes
4. **Optimize images** and content for better performance
5. **Set up version control** with Git
6. **Deploy to cloud** using provided configurations

## Features Included

✅ **Responsive Design** - Works on all devices
✅ **Dark/Light Theme** - Toggle button included
✅ **Smooth Animations** - AOS and custom animations
✅ **SEO Optimized** - Meta tags and structured data
✅ **PWA Ready** - Service worker included
✅ **Performance Optimized** - Lazy loading and caching
✅ **Accessibility** - ARIA labels and keyboard navigation
✅ **Modern Stack** - Vite, ES6+, CSS Grid/Flexbox

## Support

If you encounter any issues:

1. Check the troubleshooting section above
2. Ensure all prerequisites are installed
3. Try clearing cache and reinstalling dependencies
4. Check the browser console for error messages

## Development Workflow

```bash
# Daily development workflow:
cd your-project-folder
npm run dev

# Make your changes in:
# - index.html (content)
# - src/styles.css (styling)
# - src/main.js (functionality)

# Test your changes:
npm test

# Build for production:
npm run build
npm run preview
```

Happy coding! 🚀