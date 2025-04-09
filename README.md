# Spline 3D Parameter Debugging Tool

[![Vue.js](https://img.shields.io/badge/Vue.js-v3.x-4FC08D?style=flat-square&logo=vue.js)](https://vuejs.org/)
[![Vite](https://img.shields.io/badge/Vite-v4.x-646CFF?style=flat-square&logo=vite)](https://vitejs.dev/)
[![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)](LICENSE)

> A modern 3D parameter debugging tool based on Vue 3 and Spline, providing an intuitive 3D scene control experience

![demo](https://github.com/Eins51/spline-vue-tool/blob/master/demo/demo.gif)

## ✨ Features

- **3D Scene Display**: Integrates Spline 3D engine for high-quality 3D rendering
- **Intuitive Camera Control**: Supports precise camera position adjustment in the range of -1000 to 1000
- **Background Control**: Customize background colors and gradient effects
- **Ground Control**: Change ground colors and select from multiple ground objects
- **Real-time View Ratio Adjustment**: Dynamically adjust view size and zoom level
- **Responsive Design**: Perfect adaptation for desktop and mobile devices
- **Modern UI**: Magnetic liquid title animation and elegant control panel

## 🛠️ Technology Stack

- [Vue 3](https://vuejs.org/) - Progressive JavaScript Framework
- [Vite](https://vitejs.dev/) - Next Generation Frontend Build Tool
- [Spline 3D](https://spline.design/) - 3D Design and Scene Rendering
- [GSAP](https://greensock.com/gsap/) - Professional JavaScript Animation Library
- [Vue Router](https://router.vuejs.org/) - Official Router for Vue.js

## 🚀 Quick Start

### Prerequisites

- Node.js 16.x or higher
- npm 7.x or higher

### Installation

1. Clone the repository
```bash
git clone https://github.com/yourusername/spline-3d-controller.git
cd spline-3d-controller
```

2. Install dependencies:
```bash
npm install
```

3. Run development server:
```bash
npm run dev
```

4. Build for production:
```bash
npm run build
```

## 📁 Project Structure

```
spline-3d-controller/
├── demo/                   # Demo examples
│   └── demo.gif            # Demo example
├── public/                 # Static assets
├── src/
│   ├── components/
│   │   └── VueSpline.vue   # Spline integration component
│   ├── views/
│   │   └── Home.vue        # Main view component
│   ├── App.vue             # Root component
│   ├── main.js             # Entry file
│   └── router/             # Router configuration
│       └── index.js
├── index.html              # HTML template
├── package.json            # Project dependencies
├── vite.config.js          # Vite configuration
├── LICENSE                 # License file
└── README.md               # Project documentation
```

Main files and directories explanation:

- **demo/**: Contains various practical examples to help users understand the tool's features
- **src/components/VueSpline.vue**: Encapsulates Spline loading and initialization logic
- **src/views/Home.vue**: Implements the main interface and control panel functionality

## 📖 Usage Instructions

1. **Basic Control**: Adjust view ratio and examine scene structure, with ability to copy API information and scene structure directly in the interface
2. **Background Control**: Select preset colors from the palette or input custom color values, and adjust gradient effects
3. **Ground Control**: Select and change the color of ground objects
4. **Camera Control**: Use sliders to precisely adjust X, Y, Z axis positions, or apply preset views

## 🔌 Spline API Integration Logic

### 1. API Integration Architecture

The project adopts a layered structure to integrate the Spline API:

- **VueSpline Component**: Encapsulates Spline loading and initialization logic
- **Home Component**: Implements specific interaction features and UI controls
- **Global State**: Uses Vue's reactive system to manage state

### 2. Initialization Process

```javascript
// Define scene URL
const splineUrl = "https://prod.spline.design/PBQQBw8bfXDhBo7w/scene.splinecode";

// Spline load event handler
const onSplineLoad = (app) => {
  splineApp = app;
  
  // Initial settings
  app.setBackgroundColor('transparent');
  setZoom(cameraDistance.value / 100);
  
  // Get camera position
  if (app._camera) {
    const camera = app._camera;
    // Get current position
    const currentPos = {
      x: camera.position.x,
      y: camera.position.y,
      z: camera.position.z
    };
    
    // Set initial camera position
    cameraPosition.x = currentPos.x || defaultCameraPosition.x;
    cameraPosition.y = currentPos.y || defaultCameraPosition.y;
    cameraPosition.z = currentPos.z || defaultCameraPosition.z;
  }
  
  // Find scene objects
  findGroundObjects();
};
```

### 3. API Call Patterns

#### 3.1 Public API Calls

Spline provides some public API methods that can be called directly:

```javascript
// Set background color
splineApp.setBackgroundColor(color);

// Update scene
splineApp.update();

// Set zoom
splineApp.setZoom(value);

// Query scene information
const objectCount = splineApp.countObjects();
```

#### 3.2 Internal API/Property Access

By accessing Spline's internal properties, more advanced control can be achieved:

```javascript
// Access camera
const camera = splineApp._camera;
camera.position.set(x, y, z);
camera.lookAt(0, 0, 0);

// Access scene
const scene = splineApp._scene;
const activePage = scene.activePage;

// Access renderer
const renderer = splineApp._renderer;
renderer.render(scene, camera);

// Change material properties
const material = object.material;
material.color.set(newColor);
material.opacity = 0.5;
material.transparent = true;
material.needsUpdate = true;
```

### 4. Main Functional Modules

#### 4.1 Scene Object Finding and Traversal

The project implements automatic finding of objects in the scene, especially for ground/floor objects:

```javascript
// Find potential ground/floor objects
const findGroundObjects = () => {
  groundObjects.value = [];
  
  try {
    // Get active page
    const activePage = splineApp._scene?.activePage;
    
    if (activePage) {
      // Traverse scene looking for ground objects
      activePage.traverse((object) => {
        // Check if object has material
        if (object.material) {
          const objName = (object.name || '').toLowerCase();
          
          // Determine if it's a ground based on name or characteristics
          const isNameMatch = objName.includes('ground') || 
            objName.includes('floor') || 
            objName.includes('plane');
          
          // Add to ground objects list
          if (isNameMatch || object.type === 'Mesh') {
            groundObjects.value.push({
              uuid: object.uuid,
              name: object.name || 'Unnamed Ground',
              type: object.type,
              object: object
            });
            
            // Store material reference
            groundMaterials.set(object.uuid, object.material);
          }
        }
      });
    }
  } catch (err) {
    console.error('Error searching for ground objects:', err);
  }
};

// Find all objects and build hierarchy structure
const findSplineObjects = () => {
  // Implementation logic...
};

// Recursively traverse scene nodes
const traverseScene = (node, sceneData) => {
  // Implementation logic...
};
```

#### 4.2 Camera Control

```javascript
// Update camera position
const updateCameraPosition = () => {
  if (!splineApp || !splineApp._camera) return;
  
  try {
    const camera = splineApp._camera;
    camera.position.set(cameraPosition.x, cameraPosition.y, cameraPosition.z);
    
    if (typeof camera.lookAt === 'function') {
      camera.lookAt(0, 0, 0); // Look at origin
    }
    
    if (typeof splineApp.update === 'function') {
      splineApp.update();
    }
  } catch (err) {
    console.error('Error updating camera position:', err);
  }
};

// Reset camera position
const resetCameraPosition = () => {
  cameraPosition.x = defaultCameraPosition.x;
  cameraPosition.y = defaultCameraPosition.y;
  cameraPosition.z = defaultCameraPosition.z;
  updateCameraPosition();
};

// Jump to specified position
const jumpToPosition = () => {
  cameraPosition.x = manualPosition.x;
  cameraPosition.y = manualPosition.y;
  cameraPosition.z = manualPosition.z;
  updateCameraPosition();
};

// Apply preset position
const applyPresetPosition = (preset) => {
  cameraPosition.x = preset.x;
  cameraPosition.y = preset.y;
  cameraPosition.z = preset.z;
  updateCameraPosition();
};
```

#### 4.3 Material Control

```javascript
// Change ground color
const changeGroundColor = (color) => {
  if (!splineApp || !selectedGroundObject.value) return;
  
  try {
    const targetObjectInfo = groundObjects.value.find(
      obj => obj.uuid === selectedGroundObject.value
    );
    
    if (!targetObjectInfo?.object?.material) return;
    
    const targetObject = targetObjectInfo.object;
    
    if (targetObject.material.color) {
      if (color === 'transparent') {
        targetObject.material.color.set('#333333');
        targetObject.material.opacity = 0.5;
        targetObject.material.transparent = true;
      } else {
        targetObject.material.color.set(color);
        targetObject.material.opacity = 1;
        targetObject.material.transparent = false;
      }
      
      targetObject.material.needsUpdate = true;
    }
    
    // Force render update
    if (typeof splineApp.update === 'function') {
      splineApp.update();
    }
  } catch (err) {
    console.error('Error setting ground color:', err);
  }
};

// Change background color
const changeBackground = (color) => {
  if (!splineApp) return;
  
  try {
    // Set base background color via API
    if (typeof splineApp.setBackgroundColor === 'function') {
      splineApp.setBackgroundColor(color);
    }
    
    // Apply CSS gradient effect
    const canvas = splineApp._renderer?.domElement;
    if (canvas) {
      canvas.style.background = `linear-gradient(${color} 0%, transparent ${gradientStop.value * 100}%)`;
    }
    
    // Force update
    if (typeof splineApp.update === 'function') {
      splineApp.update();
    }
  } catch (err) {
    console.error('Error setting background color:', err);
  }
};

// Update gradient
const updateGradient = () => {
  if (customBgColor.value && splineApp) {
    changeBackground(customBgColor.value);
  }
};
```

#### 4.4 Zoom Control

```javascript
// Set zoom ratio
const setZoom = (value) => {
  if (!splineApp) return;
  
  try {
    // Method 1: Scale using canvas style
    const canvas = splineApp.domElement;
    if (canvas) {
      canvas.style.transform = `scale(${value})`;
    }
    
    // Method 2: Use internal API to control view
    if (typeof splineApp.setZoom === 'function') {
      splineApp.setZoom(value);
    }
    
    // Force render update
    if (typeof splineApp.update === 'function') {
      splineApp.update();
    }
  } catch (err) {
    console.error('Error setting zoom:', err);
  }
};

// Update camera distance
const updateCameraDistance = () => {
  setZoom(cameraDistance.value / 100);
};

// Scale specific object
const scaleObject = (obj, scale) => {
  if (!obj || !obj.scale) return;
  
  try {
    obj.scale.set(scale, scale, scale);
    
    // Force scene update
    if (splineApp && typeof splineApp.update === 'function') {
      splineApp.update();
    }
  } catch (err) {
    console.error('Error scaling object:', err);
  }
};
```

### 5. Error Handling Strategies

The project uses two main error handling strategies:

1. **Condition Checking**: Check if methods exist before calling them
   ```javascript
   // Check if method exists before calling
   if (typeof splineApp.setZoom === 'function') {
     splineApp.setZoom(value);
   }
   
   // Check if object exists before accessing properties
   if (splineApp && splineApp._camera) {
     // Safely access camera
   }
   ```

2. **Exception Catching**: Wrap potentially error-prone operations with try-catch
   ```javascript
   try {
     setZoom(cameraDistance.value / 100);
   } catch (err) {
     console.error('Error setting zoom:', err);
   }
   ```

### 6. API Exploration Feature

The project provides an API information display feature to view the structure of the Spline API in the interface:

```javascript
// Display API information
const showSplineAPI = () => {
  if (!splineApp) return;
  
  apiInfo.value = '';
  const apiData = {};
  
  // Collect API information
  apiData.properties = Object.keys(splineApp);
  apiData.methods = Object.getOwnPropertyNames(Object.getPrototypeOf(splineApp))
    .filter(prop => typeof splineApp[prop] === 'function' && prop !== 'constructor');
  
  // Check key properties
  const keyProps = ['_scene', '_camera', '_renderer', '_data', 'domElement'];
  apiData.keyComponents = {};
  
  keyProps.forEach(prop => {
    if (splineApp[prop]) {
      apiData.keyComponents[prop] = {
        type: Object.prototype.toString.call(splineApp[prop]),
        properties: splineApp[prop] ? Object.keys(splineApp[prop]) : []
      };
    }
  });
  
  // Display information dialog
  apiInfo.value = JSON.stringify(apiData, null, 2);
  showApiDialog.value = true;
};
```

## 📜 License

This project is released under the MIT License - see the [LICENSE](LICENSE) file for details

## 🔗 Related Links

- [Spline Official Website](https://spline.design/)
- [Vue 3 Documentation](https://vuejs.org/)

