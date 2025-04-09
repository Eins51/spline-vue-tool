<template>
    <div class="ball-test">
      <div class="title-container">
        <div class="magnetic-title">
          <h1 ref="headerTitle" class="liquid-title">
            <span class="text-wrapper">
              <span class="letters">Spline 3D 参数调试工具</span>
              <span class="line"></span>
            </span>
          </h1>
        </div>
      </div>
      
      <div class="content-container">
        <div class="ball-container">
          <vue-spline 
            :url="splineUrl"
            background="transparent"
            :containerStyle="{ background: 'transparent' }"
            :canvasStyle="{ background: 'transparent' }"
            :options="{
              zoom: 0.3,
              objectScale: { x: 0.05, y: 0.05, z: 0.05 },
              cameraPosition: { x: 0, y: 0, z: 5 }
            }"
            @load="onSplineLoad"
          />
        </div>
      </div>
      
      <!-- 控制面板改为底部 -->
      <div class="control-panel">
        <div class="panel-tabs">
          <button 
            class="tab-btn" 
            :class="{ active: activeTab === 'basic' }" 
            @click="activeTab = 'basic'"
          >
            基本控制
          </button>
          <button 
            class="tab-btn" 
            :class="{ active: activeTab === 'background' }" 
            @click="activeTab = 'background'"
          >
            背景控制
          </button>
          <button 
            class="tab-btn" 
            :class="{ active: activeTab === 'ground' }" 
            @click="activeTab = 'ground'"
          >
            场地控制
          </button>
          <button 
            class="tab-btn" 
            :class="{ active: activeTab === 'camera' }" 
            @click="activeTab = 'camera'"
          >
            相机控制
          </button>
        </div>
        
        <!-- 基本控制选项卡 -->
        <div class="panel-content" v-show="activeTab === 'basic'">
          <div class="control-section">
            <!-- 视图滑块 -->
            <div class="slider-container">
              <label>视图比例: {{ (cameraDistance / 100).toFixed(2) }}</label>
              <input type="range" v-model.number="cameraDistance" min="5" max="100" step="5" @input="updateCameraDistance" />
            </div>
            
            <!-- 按钮组 -->
            <div class="button-group">
              <button @click="updateCameraDistance" class="primary-btn">应用视图比例</button>
              <button @click="findSplineObjects" class="secondary-btn">分析场景结构</button>
              <button @click="showSplineAPI" class="secondary-btn">打印API信息</button>
            </div>
          </div>
        </div>
        
        <!-- 背景控制选项卡 -->
        <div class="panel-content" v-show="activeTab === 'background'">
          <div class="control-section">
            <h3>背景控制:</h3>
            
            <!-- 颜色选择区 -->
            <div class="color-buttons">
              <button 
                v-for="color in colorOptions" 
                :key="color.name"
                class="color-btn"
                :style="{ backgroundColor: color.value }"
                @click="changeBackground(color.value)"
                :title="color.name"
              ></button>
            </div>
            
            <!-- 自定义颜色 -->
            <div class="custom-color">
              <input type="text" v-model="customBgColor" placeholder="#000000">
              <button @click="changeBackground(customBgColor)" class="apply-btn">应用</button>
            </div>
            
            <!-- 渐变控制 -->
            <div class="gradient-slider">
              <span>渐变停止点: {{ gradientStop }}</span>
              <input 
                type="range" 
                min="0.1" 
                max="1" 
                step="0.1" 
                v-model="gradientStop" 
                @input="updateGradient"
              />
            </div>
          </div>
        </div>
        
        <!-- 场地颜色控制选项卡 -->
        <div class="panel-content" v-show="activeTab === 'ground'">
          <div class="control-section">
            <h3>场地颜色控制:</h3>
            
            <!-- 场地颜色选择区 -->
            <div class="color-buttons">
              <button 
                v-for="color in colorOptions" 
                :key="color.name"
                class="color-btn"
                :style="{ backgroundColor: color.value }"
                @click="changeGroundColor(color.value)"
                :title="color.name"
              ></button>
            </div>
            
            <!-- 自定义场地颜色 -->
            <div class="custom-color">
              <input type="text" v-model="customGroundColor" placeholder="#00FF00">
              <button @click="changeGroundColor(customGroundColor)" class="apply-btn">应用</button>
            </div>
            
            <!-- 场地对象选择 -->
            <div class="object-selector" v-if="groundObjects.length > 0">
              <label>选择场地对象:</label>
              <select v-model="selectedGroundObject" @change="onGroundObjectChange">
                <option v-for="obj in groundObjects" :key="obj.uuid" :value="obj.uuid">
                  {{ obj.name }} ({{ obj.type }})
                </option>
              </select>
            </div>
          </div>
        </div>
        
        <!-- 相机位置控制选项卡 -->
        <div class="panel-content" v-show="activeTab === 'camera'">
          <div class="control-section">
            <h3>相机位置控制:</h3>
            
            <!-- X轴位置控制 -->
            <div class="slider-container">
              <label>X轴位置: {{ cameraPosition.x.toFixed(2) }}</label>
              <input 
                type="range" 
                v-model.number="cameraPosition.x" 
                min="-1000" 
                max="1000" 
                step="10" 
                @input="updateCameraPosition" 
              />
            </div>
            
            <!-- Y轴位置控制 -->
            <div class="slider-container">
              <label>Y轴位置: {{ cameraPosition.y.toFixed(2) }}</label>
              <input 
                type="range" 
                v-model.number="cameraPosition.y" 
                min="-1000" 
                max="1000" 
                step="10" 
                @input="updateCameraPosition" 
              />
            </div>
            
            <!-- Z轴位置控制 -->
            <div class="slider-container">
              <label>Z轴位置: {{ cameraPosition.z.toFixed(2) }}</label>
              <input 
                type="range" 
                v-model.number="cameraPosition.z" 
                min="-1000" 
                max="1000" 
                step="10" 
                @input="updateCameraPosition" 
              />
            </div>
            
            <!-- 相机控制按钮 -->
            <div class="button-group">
              <button @click="resetCameraPosition" class="primary-btn">重置相机位置</button>
            </div>
            
            <!-- 手动设置相机位置 -->
            <div class="manual-position-set">
              <h4>快速定位</h4>
              <div class="position-inputs">
                <div class="position-input-group">
                  <label>X:</label>
                  <input type="number" v-model.number="manualPosition.x" class="position-input" />
                </div>
                <div class="position-input-group">
                  <label>Y:</label>
                  <input type="number" v-model.number="manualPosition.y" class="position-input" />
                </div>
                <div class="position-input-group">
                  <label>Z:</label>
                  <input type="number" v-model.number="manualPosition.z" class="position-input" />
                </div>
                <button @click="jumpToPosition" class="primary-btn">跳转</button>
              </div>
              
              <!-- 预设位置 -->
              <div class="preset-positions">
                <h4>预设位置</h4>
                <div class="preset-buttons">
                  <button 
                    v-for="(preset, index) in presetPositions" 
                    :key="index" 
                    @click="applyPresetPosition(preset)"
                    class="preset-btn"
                  >
                    {{ preset.name }}
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- API信息弹窗 -->
      <div v-if="showApiDialog" class="dialog-overlay" @click.self="showApiDialog = false">
        <div class="dialog-content">
          <div class="dialog-header">
            <h2>Spline API 信息</h2>
            <div class="dialog-actions">
              <button @click="copyApiInfo" class="dialog-btn copy-btn">复制</button>
              <button @click="showApiDialog = false" class="dialog-btn close-btn">关闭</button>
            </div>
          </div>
          <div class="dialog-body">
            <pre>{{ apiInfo }}</pre>
          </div>
        </div>
      </div>

      <!-- 场景结构弹窗 -->
      <div v-if="showSceneDialog" class="dialog-overlay" @click.self="showSceneDialog = false">
        <div class="dialog-content">
          <div class="dialog-header">
            <h2>场景结构信息</h2>
            <div class="dialog-actions">
              <button @click="copySceneInfo" class="dialog-btn copy-btn">复制</button>
              <button @click="showSceneDialog = false" class="dialog-btn close-btn">关闭</button>
            </div>
          </div>
          <div class="dialog-body">
            <pre>{{ sceneInfo }}</pre>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, reactive, computed, onMounted } from 'vue';
  import VueSpline from '@/components/VueSpline.vue';
  import { gsap } from 'gsap';
  
  // DOM引用
  const headerTitle = ref(null);
  
  // ---------------------------------------------
  // 基础配置和状态
  // ---------------------------------------------
  const splineUrl = "https://prod.spline.design/PBQQBw8bfXDhBo7w/scene.splinecode";
  const scaleValue = ref(0.5);
  const cameraDistance = ref(30);
  const activeTab = ref('basic'); // 默认选中基本控制选项卡
  let splineApp = null;
  
  // 场景对象相关状态
  const sceneObjects = ref([]);
  const groundObjects = ref([]);
  const groundMaterials = new Map(); // 存储场地材质引用
  const selectedGroundObject = ref(null);
  
  // 颜色相关状态
  const customBgColor = ref("#000000");
  const customGroundColor = ref("#00FF00");
  const gradientStop = ref(0.7);
  
  // 弹窗相关状态
  const showApiDialog = ref(false);
  const showSceneDialog = ref(false);
  const apiInfo = ref('');
  const sceneInfo = ref('');
  
  // 预定义的颜色选项
  const colorOptions = [
    { name: '透明', value: 'transparent' },
    { name: '白色', value: '#FFFFFF' },
    { name: '黑色', value: '#000000' },
    { name: '靛蓝', value: '#4A5568' },
    { name: '薄荷', value: '#48BB78' },
    { name: '青绿', value: '#38B2AC' },
    { name: '玫瑰', value: '#ED64A6' },
    { name: '紫罗兰', value: '#805AD5' }
  ];
  
  // ---------------------------------------------
  // 相机位置控制相关状态
  // ---------------------------------------------
  // 相机位置控制
  const cameraPosition = reactive({
    x: 0,
    y: 0,
    z: 300
  });
  
  // 手动输入的相机位置
  const manualPosition = reactive({
    x: 0,
    y: 0,
    z: 300
  });
  
  // 初始相机位置（用于重置）
  const defaultCameraPosition = {
    x: 0,
    y: 0,
    z: 300
  };
  
  // 预设的相机位置
  const presetPositions = [
    { name: '正前方', x: 0, y: 0, z: 800 },
    { name: '侧面45°', x: 500, y: 0, z: 800 },
    { name: '上方俯视', x: 0, y: 500, z: 800 },
    { name: '左侧面', x: -500, y: 0, z: 800 },
    { name: '右侧面', x: 500, y: 0, z: 800 },
    { name: '右上45°', x: 500, y: 500, z: 800 }
  ];
  
  // ---------------------------------------------
  // Spline 初始化和加载
  // ---------------------------------------------
  // 处理Spline加载完成事件
  const onSplineLoad = (app) => {
    console.log('Spline scene loaded in test page:', app);
    splineApp = app;
    
    // 设置背景为透明
    if (typeof app.setBackgroundColor === 'function') {
      app.setBackgroundColor('transparent');
      console.log('已设置背景为透明');
    }
    
    // 设置默认相机距离
    try {
      setZoom(cameraDistance.value / 100);
    } catch (err) {
      console.error('设置默认相机距离失败:', err);
    }
    
    // 如果存在相机，获取当前相机位置
    if (app._camera) {
      try {
        const camera = app._camera;
        // 获取当前位置
        const currentPos = {
          x: camera.position.x,
          y: camera.position.y,
          z: camera.position.z
        };
        console.log('当前相机位置:', currentPos);
        
        // 如果当前位置太近，设置为默认位置
        if (Math.abs(currentPos.z) < 10) {
          cameraPosition.x = defaultCameraPosition.x;
          cameraPosition.y = defaultCameraPosition.y;
          cameraPosition.z = defaultCameraPosition.z;
          updateCameraPosition();
        } else {
          // 否则使用当前位置
          cameraPosition.x = currentPos.x;
          cameraPosition.y = currentPos.y;
          cameraPosition.z = currentPos.z;
        }
        console.log('设置相机位置为:', cameraPosition);
      } catch (err) {
        console.error('获取相机位置失败:', err);
      }
    }
    
    // 自动查找场景中的地板/场地对象
    findGroundObjects();
  };
  
  // 显示Spline API信息
  const showSplineAPI = () => {
    if (!splineApp) return;
    
    apiInfo.value = '';
    const apiData = {};
    
    // 收集API信息
    apiData.properties = Object.keys(splineApp);
    apiData.methods = Object.getOwnPropertyNames(Object.getPrototypeOf(splineApp))
      .filter(prop => typeof splineApp[prop] === 'function' && prop !== 'constructor');
    
    // 检查关键属性
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
    
    // 格式化API信息
    apiInfo.value = JSON.stringify(apiData, null, 2);
    
    // 显示弹窗
    showApiDialog.value = true;
  };
  
  // 复制API信息
  const copyApiInfo = () => {
    navigator.clipboard.writeText(apiInfo.value)
      .then(() => {
        alert('API信息已复制到剪贴板');
      })
      .catch(err => {
        console.error('复制失败:', err);
        alert('复制失败，请手动选择并复制');
      });
  };
  
  // ---------------------------------------------
  // 场景对象查找与遍历
  // ---------------------------------------------
  // 查找场景中可能是地板/场地的对象
  const findGroundObjects = () => {
    if (!splineApp) return;
    
    groundObjects.value = [];
    groundMaterials.clear();
    
    try {
      // 获取活动页面（包含实际场景内容）
      const activePage = splineApp._scene?.activePage;
      
      if (activePage) {
        console.log('正在搜索场地对象...');
        
        // 遍历场景查找可能的地板/场地对象
        activePage.traverse((object) => {
          // 检查对象是否有材质
          if (object.material) {
            const objName = (object.name || '').toLowerCase();
            
            // 如果对象名称包含关键词或是平面，可能是地板
            const isNameMatch = 
              objName.includes('ground') || 
              objName.includes('floor') || 
              objName.includes('plane');
            
            // 检查是否是大的扁平对象（可能是地板）
            let isFlat = false;
            if (object.geometry && object.geometry.boundingBox) {
              const box = object.geometry.boundingBox;
              const height = Math.abs(box.max.y - box.min.y);
              const width = Math.abs(box.max.x - box.min.x);
              const depth = Math.abs(box.max.z - box.min.z);
              
              // 如果高度很小但宽度和深度较大，可能是地板
              isFlat = height < 0.1 * Math.max(width, depth);
            }
            
            // 查看材质颜色是否为绿色或与此图片中的绿色接近
            let isGreenish = false;
            if (object.material.color) {
              const color = object.material.color;
              isGreenish = color.g > 0.5 && color.r < 0.5 && color.b < 0.5;
            }
            
            // 检查对象的位置是否在底部
            const isLowPosition = object.position.y < 0;
            
            // 如果符合任一条件，添加到地板对象列表
            if (isNameMatch || isFlat || isGreenish || isLowPosition || object.type === 'Mesh') {
              console.log('找到可能的场地对象:', object.name || '未命名', object);
              
              groundObjects.value.push({
                uuid: object.uuid,
                name: object.name || '未命名场地',
                type: object.type,
                object: object
              });
              
              groundMaterials.set(object.uuid, object.material);
              
              // 如果还没有选中的场地对象，则选择第一个找到的
              if (!selectedGroundObject.value) {
                selectedGroundObject.value = object.uuid;
              }
            }
          }
        });
        
        console.log('找到', groundObjects.value.length, '个可能的场地对象');
      }
    } catch (err) {
      console.error('搜索场地对象时出错:', err);
    }
  };
  
  // 查找场景中的对象
  const findSplineObjects = () => {
    if (!splineApp) return;
    
    sceneObjects.value = [];
    sceneInfo.value = '';
    
    try {
      const sceneData = {
        objects: [],
        structure: {}
      };
      
      // 检查_scene属性
      if (splineApp._scene) {
        console.log('找到内部_scene属性:', splineApp._scene);
        traverseScene(splineApp._scene, sceneData);
      }
      
      // 检查renderer的scene
      if (splineApp._renderer && splineApp._renderer.scene) {
        console.log('找到渲染器场景:', splineApp._renderer.scene);
        traverseScene(splineApp._renderer.scene, sceneData);
      }
      
      // 格式化场景信息
      sceneInfo.value = JSON.stringify(sceneData, null, 2);
      
      // 显示弹窗
      showSceneDialog.value = true;
      
      // 顺便也查找地板/场地对象
      findGroundObjects();
    } catch (err) {
      console.error('无法访问场景对象:', err);
      sceneInfo.value = JSON.stringify({ error: err.message }, null, 2);
      showSceneDialog.value = true;
    }
  };
  
  // 复制场景结构信息
  const copySceneInfo = () => {
    navigator.clipboard.writeText(sceneInfo.value)
      .then(() => {
        alert('场景结构信息已复制到剪贴板');
      })
      .catch(err => {
        console.error('复制失败:', err);
        alert('复制失败，请手动选择并复制');
      });
  };
  
  // 递归遍历场景对象
  const traverseScene = (node, sceneData) => {
    // 记录当前节点
    if (node.type && node.name) {
      console.log(`发现对象: ${node.name || '未命名'} (${node.type})`);
      sceneObjects.value.push(node);
      
      // 记录到场景数据
      const objectInfo = {
        name: node.name || '未命名',
        type: node.type,
        uuid: node.uuid || '无UUID',
        position: node.position ? { 
          x: node.position.x, 
          y: node.position.y, 
          z: node.position.z 
        } : null,
        visible: node.visible,
        childCount: node.children ? node.children.length : 0
      };
      
      // 如果有材质，添加材质信息
      if (node.material) {
        objectInfo.material = {
          type: node.material.type,
          color: node.material.color ? {
            r: node.material.color.r,
            g: node.material.color.g,
            b: node.material.color.b
          } : null
        };
      }
      
      sceneData.objects.push(objectInfo);
      
      // 构建层级结构
      if (!sceneData.structure[node.uuid]) {
        sceneData.structure[node.uuid] = {
          info: objectInfo,
          children: []
        };
      }
    }
    
    // 遍历子节点
    if (node.children && node.children.length > 0) {
      node.children.forEach(child => {
        traverseScene(child, sceneData);
        
        // 添加到父节点的子节点列表
        if (node.uuid && child.uuid && sceneData.structure[node.uuid]) {
          sceneData.structure[node.uuid].children.push(child.uuid);
        }
      });
    }
  };
  
  // ---------------------------------------------
  // 颜色和材质控制
  // ---------------------------------------------
  // 当选择的场地对象改变时
  const onGroundObjectChange = () => {
    if (customGroundColor.value) {
      changeGroundColor(customGroundColor.value);
    }
  };
  
  // 更改场地颜色
  const changeGroundColor = (color) => {
    console.log('尝试设置场地颜色:', color);
    
    // 更新自定义场地颜色状态
    customGroundColor.value = color;
    
    if (!splineApp || !selectedGroundObject.value) {
      console.error('无法设置场地颜色: Spline应用未加载或未选择场地对象');
      return;
    }
    
    try {
      // 获取选中的场地对象
      const targetObjectInfo = groundObjects.value.find(obj => obj.uuid === selectedGroundObject.value);
      
      if (!targetObjectInfo || !targetObjectInfo.object || !targetObjectInfo.object.material) {
        console.error('选中的对象没有材质');
        return;
      }
      
      const targetObject = targetObjectInfo.object;
      console.log('正在更改场地对象材质颜色:', targetObject.name || '未命名');
      
      // 如果材质有颜色属性，直接修改
      if (targetObject.material.color) {
        // 如果是透明色，使用浅灰色
        if (color === 'transparent') {
          targetObject.material.color.set('#333333');
          targetObject.material.opacity = 0.5;
          targetObject.material.transparent = true;
        } else {
          targetObject.material.color.set(color);
          targetObject.material.opacity = 1;
          targetObject.material.transparent = false;
        }
        
        // 标记材质需要更新
        targetObject.material.needsUpdate = true;
        console.log('成功更改场地颜色为:', color);
      }
      
      // 强制更新渲染
      if (typeof splineApp.update === 'function') {
        splineApp.update();
      } else if (splineApp._renderer && typeof splineApp._renderer.render === 'function') {
        splineApp._renderer.render(splineApp._scene, splineApp._camera);
      }
    } catch (err) {
      console.error('设置场地颜色时出错:', err);
    }
  };
  
  // 更改背景颜色
  const changeBackground = (color) => {
    console.log('尝试设置背景颜色:', color);
    
    // 更新自定义背景颜色状态
    customBgColor.value = color;
    
    if (!splineApp) {
      console.error('Spline应用程序尚未加载，无法设置背景');
      return;
    }
    
    try {
      // 通过API设置基础背景色
      if (typeof splineApp.setBackgroundColor === 'function') {
        splineApp.setBackgroundColor(color);
        console.log('使用API方法设置基础背景色:', color);
      }
      
      // 应用CSS渐变效果 - 通过_renderer.domElement访问canvas
      const canvas = splineApp._renderer?.domElement;
      if (canvas) {
        canvas.style.background = `linear-gradient(${color} 0%, transparent ${gradientStop.value * 100}%)`;
        console.log('通过CSS设置背景渐变:', color, '渐变停止点:', gradientStop.value);
      }
      
      // 强制更新
      if (typeof splineApp.update === 'function') {
        splineApp.update();
      }
    } catch (err) {
      console.error('设置背景颜色时出错:', err);
    }
  };
  
  // 更新渐变
  const updateGradient = () => {
    if (customBgColor.value && splineApp) {
      changeBackground(customBgColor.value);
    }
  }
  
  // ---------------------------------------------
  // 缩放和比例控制
  // ---------------------------------------------
  // 缩放特定对象
  const scaleObject = (obj, scale) => {
    if (!obj || !obj.scale) return;
    
    try {
      obj.scale.set(scale, scale, scale);
      
      // 强制场景更新
      if (splineApp && typeof splineApp.update === 'function') {
        splineApp.update();
      }
    } catch (err) {
      console.error('缩放对象时出错:', err);
    }
  };
  
  // 设置缩放比例
  const setZoom = (value) => {
    if (!splineApp) return;
    
    try {
      // 方法1: 使用canvas样式缩放
      const canvas = splineApp.domElement;
      if (canvas) {
        canvas.style.transform = `scale(${value})`;
      }
      
      // 方法2: 检查是否有内部API可以控制视图
      if (typeof splineApp.setZoom === 'function') {
        splineApp.setZoom(value);
      }
      
      // 强制更新渲染
      if (typeof splineApp.update === 'function') {
        splineApp.update();
      }
    } catch (err) {
      console.error('设置缩放失败:', err);
    }
  };
  
  // 更新相机距离
  const updateCameraDistance = () => {
    setZoom(cameraDistance.value / 100);
  };
  
  // ---------------------------------------------
  // 相机位置控制
  // ---------------------------------------------
  // 更新相机位置
  const updateCameraPosition = () => {
    if (!splineApp || !splineApp._camera) {
      console.error('无法更新相机位置：相机不可用');
      return;
    }
    
    try {
      // 获取相机引用
      const camera = splineApp._camera;
      
      // 设置相机位置
      camera.position.set(cameraPosition.x, cameraPosition.y, cameraPosition.z);
      
      // 如果相机有lookAt方法，可以使其始终看向原点或特定对象
      if (typeof camera.lookAt === 'function') {
        camera.lookAt(0, 0, 0); // 注视原点
      }
      
      // 强制更新渲染
      if (typeof splineApp.update === 'function') {
        splineApp.update();
      } else if (splineApp._renderer && typeof splineApp._renderer.render === 'function') {
        splineApp._renderer.render(splineApp._scene, camera);
      }
      
      console.log('相机位置已更新:', cameraPosition);
    } catch (err) {
      console.error('更新相机位置时出错:', err);
    }
  };
  
  // 重置相机位置
  const resetCameraPosition = () => {
    // 重置为默认值
    cameraPosition.x = defaultCameraPosition.x;
    cameraPosition.y = defaultCameraPosition.y;
    cameraPosition.z = defaultCameraPosition.z;
    
    // 应用更新
    updateCameraPosition();
  };
  
  // 跳转到手动输入的位置
  const jumpToPosition = () => {
    // 将手动输入的位置复制到当前相机位置
    cameraPosition.x = manualPosition.x;
    cameraPosition.y = manualPosition.y;
    cameraPosition.z = manualPosition.z;
    
    // 更新相机位置
    updateCameraPosition();
  };
  
  // 应用预设位置
  const applyPresetPosition = (preset) => {
    // 将预设位置复制到当前相机位置
    cameraPosition.x = preset.x;
    cameraPosition.y = preset.y;
    cameraPosition.z = preset.z;
    
    // 同时更新手动输入框中的值
    manualPosition.x = preset.x;
    manualPosition.y = preset.y;
    manualPosition.z = preset.z;
    
    // 更新相机位置
    updateCameraPosition();
  };
  
  // 标题动画
  onMounted(() => {
    if (headerTitle.value) {
      // 初始化磁性效果
      initMagneticEffect();
      
      // 初始化文字动画
      initTextAnimation();
    }
  });
  
  // 初始化磁性效果
  function initMagneticEffect() {
    const title = document.querySelector('.magnetic-title');
    const letters = document.querySelectorAll('.liquid-title .letters');
    
    // 跟踪鼠标位置
    window.addEventListener('mousemove', (e) => {
      if (!title) return;
      
      const rect = title.getBoundingClientRect();
      const titleCenterX = rect.left + rect.width / 2;
      const titleCenterY = rect.top + rect.height / 2;
      
      // 计算鼠标与标题中心的距离
      const distanceX = e.clientX - titleCenterX;
      const distanceY = e.clientY - titleCenterY;
      
      // 检查鼠标是否在标题附近
      const distance = Math.sqrt(distanceX * distanceX + distanceY * distanceY);
      const maxDistance = Math.max(rect.width, rect.height) * 1.5;
      
      if (distance < maxDistance) {
        // 计算磁性效果强度，距离越近效果越强
        const strength = (maxDistance - distance) / maxDistance;
        const moveX = distanceX * strength * 0.3;
        const moveY = distanceY * strength * 0.2;
        
        // 应用变换
        gsap.to(title, {
          x: moveX,
          y: moveY,
          rotation: moveX * 0.05,
          duration: 0.6,
          ease: "power2.out"
        });
        
        // 字母扭曲效果
        letters.forEach((letter, i) => {
          const factor = (i % 2 === 0) ? 1 : -1;
          gsap.to(letter, {
            scale: 1 + 0.05 * strength,
            letterSpacing: `${strength * factor * 3}px`,
            duration: 0.6,
            ease: "power2.out"
          });
        });
        
        // 波动线条效果
        gsap.to('.line', {
          width: `${80 + strength * 20}%`,
          scaleY: 1 + strength * 1.5,
          duration: 0.4,
          ease: "elastic.out(1, 0.3)"
        });
      } else {
        // 复位效果
        gsap.to(title, {
          x: 0,
          y: 0,
          rotation: 0,
          duration: 0.7,
          ease: "elastic.out(1, 0.3)"
        });
        
        gsap.to(letters, {
          scale: 1,
          letterSpacing: "0px",
          duration: 0.7,
          ease: "elastic.out(1, 0.3)"
        });
        
        gsap.to('.line', {
          width: '100%',
          scaleY: 1,
          duration: 0.7,
          ease: "elastic.out(1, 0.5)"
        });
      }
    });
    
    // 点击效果
    title.addEventListener('click', () => {
      // 爆炸效果
      gsap.to(title, {
        scale: 1.1,
        duration: 0.2,
        onComplete: () => {
          gsap.to(title, {
            scale: 1,
            duration: 0.6,
            ease: "elastic.out(1, 0.3)"
          });
        }
      });
      
      // 字母散开再汇合效果
      const text = document.querySelector('.letters');
      const originalText = text.textContent;
      const chars = originalText.split('');
      
      text.innerHTML = chars.map(char => 
        `<span class="char">${char === ' ' ? '&nbsp;' : char}</span>`
      ).join('');
      
      const charElements = text.querySelectorAll('.char');
      
      gsap.fromTo(charElements, 
        {
          opacity: 1,
          scale: 1,
          y: 0
        },
        {
          opacity: 0.2,
          scale: Math.random() > 0.5 ? 3 : 0.1,
          y: (i) => (Math.random() - 0.5) * 100,
          stagger: 0.02,
          duration: 0.4,
          ease: "power2.out",
          onComplete: () => {
            gsap.to(charElements, {
              opacity: 1,
              scale: 1,
              y: 0,
              stagger: 0.02,
              duration: 0.4,
              ease: "elastic.out(1, 0.3)",
              onComplete: () => {
                // 恢复原始文本
                setTimeout(() => {
                  text.textContent = originalText;
                }, 500);
              }
            });
          }
        }
      );
      
      // 线条弹跳
      gsap.fromTo('.line',
        { scaleY: 1 },
        { 
          scaleY: 5,
          duration: 0.2,
          yoyo: true,
          repeat: 1,
          ease: "bounce.out" 
        }
      );
    });
  }
  
  // 文字动画初始化
  function initTextAnimation() {
    // 准备标题元素
    const text = document.querySelector('.letters');
    const line = document.querySelector('.line');
    
    // 隐藏文字和线条
    gsap.set(text, { opacity: 0, y: -40 });
    gsap.set(line, { width: 0 });
    
    // 文字渐入
    gsap.to(text, {
      opacity: 1,
      y: 0,
      duration: 1.2,
      ease: "power4.out",
      delay: 0.5
    });
    
    // 线条展开
    gsap.to(line, {
      width: '100%',
      duration: 1,
      ease: "power2.inOut",
      delay: 1.3
    });
    
    // 创建液态波浪效果
    createLiquidEffect();
  }
  
  // 创建液态波浪效果
  function createLiquidEffect() {
    const title = document.querySelector('.liquid-title');
    
    // 添加波浪容器
    const waves = document.createElement('div');
    waves.className = 'liquid-waves';
    title.appendChild(waves);
    
    // 创建多个波浪
    for (let i = 0; i < 3; i++) {
      const wave = document.createElement('div');
      wave.className = 'wave';
      wave.style.animationDelay = `${i * 0.2}s`;
      wave.style.opacity = 1 - (i * 0.2);
      waves.appendChild(wave);
    }
  }
  </script>
  
  <style>
  .ball-test {
    width: 100%;
    height: 100vh;
    position: relative;
    overflow: hidden;
    background: linear-gradient(135deg, #2b5876 0%, #4e4376 100%);
    color: white;
    display: flex;
    flex-direction: column;
  }
  
  .title-container {
    position: relative;
    text-align: center;
    padding: 35px 0;
    overflow: hidden;
    z-index: 10;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  
  .magnetic-title {
    display: inline-block;
    position: relative;
    cursor: pointer;
    transform-style: preserve-3d;
    perspective: 800px;
  }
  
  .liquid-title {
    margin: 0;
    font-size: 42px;
    font-weight: 700;
    color: #ffffff;
    position: relative;
    overflow: hidden;
    padding: 20px 40px;
  }
  
  .text-wrapper {
    position: relative;
    display: inline-block;
  }
  
  .letters {
    display: inline-block;
    position: relative;
    z-index: 2;
    background: linear-gradient(90deg, #ffffff, #88b2ea);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    text-fill-color: transparent;
    font-family: 'Poppins', sans-serif;
    letter-spacing: 0px;
    transition: letter-spacing 0.3s ease;
  }
  
  .line {
    position: absolute;
    left: 0;
    bottom: 0;
    width: 100%;
    height: 4px;
    background: linear-gradient(90deg, #5f6cef, #c476f1);
    transform-origin: bottom;
  }
  
  .liquid-waves {
    position: absolute;
    left: 0;
    right: 0;
    bottom: -15px;
    height: 20px;
    overflow: hidden;
    z-index: 1;
  }
  
  .wave {
    position: absolute;
    width: 200%;
    height: 100%;
    background: linear-gradient(90deg, rgba(95, 108, 239, 0.7), rgba(196, 118, 241, 0.7));
    border-radius: 50%;
    left: -50%;
    animation: wave 5s ease-in-out infinite alternate;
  }
  
  @keyframes wave {
    0% { transform: translateX(-25%) rotate(0deg); height: 20px; }
    50% { transform: translateX(0%) rotate(180deg); height: 25px; }
    100% { transform: translateX(25%) rotate(360deg); height: 20px; }
  }
  
  .char {
    display: inline-block;
    transition: transform 0.3s ease, opacity 0.3s ease, color 0.3s ease;
  }
  
  .content-container {
    flex: 1;
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
    overflow: hidden;
  }
  
  .ball-container {
    width: 100%;
    height: 100%;
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  
  /* 重新设计的控制面板，放在底部 */
  .control-panel {
    width: 90%;
    max-width: 1200px;
    background-color: rgba(29, 36, 71, 0.85);
    border-radius: 20px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
    position: fixed;
    left: 50%;
    bottom: 20px;
    transform: translateX(-50%);
    padding: 10px;
    z-index: 100;
    max-height: 200px;
    overflow-y: auto;
    backdrop-filter: blur(10px);
    border: 1px solid rgba(255, 255, 255, 0.1);
  }
  
  /* 选项卡样式 */
  .panel-tabs {
    display: flex;
    margin-bottom: 10px;
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
    padding-bottom: 5px;
    justify-content: center;
  }
  
  .tab-btn {
    padding: 5px 12px;
    background: rgba(95, 108, 239, 0.3);
    border: none;
    border-radius: 8px;
    color: white;
    cursor: pointer;
    margin: 0 8px;
    font-size: 13px;
    transition: all 0.2s;
    min-width: 100px;
    text-align: center;
  }
  
  .tab-btn.active {
    color: white;
    background: linear-gradient(90deg, #5f6cef, #c476f1);
    box-shadow: 0 2px 10px rgba(95, 108, 239, 0.5);
  }
  
  .tab-btn:hover {
    background: rgba(95, 108, 239, 0.5);
    transform: translateY(-2px);
  }
  
  /* 选项卡内容区 */
  .panel-content {
    padding: 0;
    height: 130px; /* 减小固定高度 */
    overflow-y: auto;
  }
  
  /* 确保所有选项卡内容具有相同高度 */
  .panel-content[v-show="activeTab === 'basic'"],
  .panel-content[v-show="activeTab === 'background'"],
  .panel-content[v-show="activeTab === 'ground'"],
  .panel-content[v-show="activeTab === 'camera'"] {
    height: 130px;
  }
  
  .panel-content h3 {
    margin: 0 0 8px 0;
    font-size: 14px;
    color: white;
    font-weight: 500;
  }
  
  .slider-container {
    background: rgba(0, 0, 0, 0.2);
    padding: 8px;
    border-radius: 10px;
    margin-bottom: 10px;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.3) inset;
  }
  
  .slider-container label {
    display: block;
    margin-bottom: 5px;
    color: rgba(255, 255, 255, 0.9);
    font-weight: 500;
    font-size: 13px;
  }
  
  input[type="range"] {
    width: 100%;
    height: 6px;
    background-color: rgba(255, 255, 255, 0.2);
    border-radius: 3px;
    -webkit-appearance: none;
  }
  
  input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 18px;
    height: 18px;
    background: linear-gradient(90deg, #5f6cef, #c476f1);
    border-radius: 50%;
    cursor: pointer;
  }
  
  .button-group {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 6px;
    margin-top: 8px;
  }
  
  .primary-btn, .secondary-btn {
    padding: 6px 10px;
    border: none;
    border-radius: 8px;
    color: white;
    cursor: pointer;
    font-size: 13px;
    transition: all 0.2s;
  }
  
  .primary-btn {
    background: linear-gradient(90deg, #5f6cef, #c476f1);
  }
  
  .primary-btn:hover {
    background: linear-gradient(90deg, #4a57d9, #b05ae0);
    transform: translateY(-2px);
    box-shadow: 0 2px 10px rgba(95, 108, 239, 0.3);
  }
  
  .secondary-btn {
    background: rgba(255, 255, 255, 0.15);
    backdrop-filter: blur(5px);
  }
  
  .secondary-btn:hover {
    background: rgba(255, 255, 255, 0.25);
    transform: translateY(-2px);
  }
  
  .color-buttons {
    display: grid;
    grid-template-columns: repeat(8, 1fr);
    gap: 5px;
    margin: 10px 0;
    justify-items: center;
    max-width: 320px;
  }
  
  .color-btn {
    width: 24px;
    height: 24px;
    border-radius: 50%;
    border: 2px solid rgba(255, 255, 255, 0.2);
    cursor: pointer;
    transition: all 0.2s;
    position: relative;
  }
  
  .color-btn:hover {
    transform: scale(1.15);
    border-color: white;
    box-shadow: 0 0 10px rgba(255, 255, 255, 0.3);
    z-index: 1;
  }
  
  /* 添加颜色提示工具 */
  .color-btn:hover::after {
    content: attr(title);
    position: absolute;
    bottom: 130%;
    left: 50%;
    transform: translateX(-50%);
    background: rgba(0, 0, 0, 0.7);
    color: white;
    padding: 3px 8px;
    border-radius: 4px;
    font-size: 11px;
    white-space: nowrap;
    pointer-events: none;
    opacity: 0.9;
  }
  
  .custom-color {
    display: flex;
    margin: 8px 0;
    background: rgba(0, 0, 0, 0.2);
    border-radius: 10px;
    padding: 6px;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.3) inset;
  }
  
  .custom-color input[type="text"] {
    flex: 1;
    padding: 6px 8px;
    background: rgba(255, 255, 255, 0.1);
    border: 1px solid rgba(255, 255, 255, 0.2);
    border-radius: 6px;
    color: white;
    margin-right: 6px;
    font-family: monospace;
    font-size: 13px;
  }
  
  .apply-btn {
    padding: 6px 10px;
    background: linear-gradient(90deg, #5f6cef, #c476f1);
    border: none;
    border-radius: 6px;
    color: white;
    cursor: pointer;
    font-weight: 500;
    transition: all 0.2s;
    font-size: 13px;
  }
  
  .apply-btn:hover {
    background: linear-gradient(90deg, #4a57d9, #b05ae0);
    transform: translateY(-2px);
    box-shadow: 0 2px 10px rgba(95, 108, 239, 0.3);
  }
  
  .gradient-slider {
    margin-top: 8px;
    background: rgba(0, 0, 0, 0.2);
    padding: 8px;
    border-radius: 10px;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.3) inset;
  }
  
  .gradient-slider span {
    display: block;
    margin-bottom: 5px;
    color: rgba(255, 255, 255, 0.9);
    font-size: 13px;
  }
  
  .object-selector {
    margin-top: 10px;
    background: rgba(0, 0, 0, 0.2);
    border-radius: 10px;
    padding: 8px;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.3) inset;
  }
  
  .object-selector label {
    display: block;
    margin-bottom: 5px;
    color: rgba(255, 255, 255, 0.9);
    font-size: 13px;
  }
  
  .object-selector select {
    width: 100%;
    padding: 6px;
    background-color: rgba(255, 255, 255, 0.1);
    border: 1px solid rgba(255, 255, 255, 0.2);
    border-radius: 6px;
    color: white;
    font-size: 13px;
  }
  
  /* 响应式设计 */
  @media (max-width: 768px) {
    .control-panel {
      width: 95%;
    }
    
    .panel-tabs {
      flex-wrap: wrap;
    }
    
    .tab-btn {
      margin: 5px;
      min-width: 80px;
    }
    
    .button-group {
      grid-template-columns: 1fr 1fr;
    }
    
    .color-buttons {
      grid-template-columns: repeat(4, 1fr);
      gap: 4px;
      max-width: 180px;
    }
  }
  
  @media (max-width: 576px) {
    .control-panel {
      max-height: 350px;
    }
    
    .tab-btn {
      min-width: 70px;
      padding: 6px 10px;
      font-size: 13px;
    }
    
    .panel-content[v-show="activeTab === 'basic'"] .slider-container,
    .panel-content[v-show="activeTab === 'basic'"] .button-group,
    .panel-content[v-show="activeTab === 'camera'"] .slider-container,
    .panel-content[v-show="activeTab === 'camera'"] .button-group,
    .color-buttons,
    .custom-color, 
    .gradient-slider, 
    .object-selector {
      min-width: 100%;
    }
    
    .color-buttons {
      grid-template-columns: repeat(4, 1fr);
      gap: 4px;
      max-width: 160px;
    }
  }
  
  @media (min-height: 700px) {
    .control-panel {
      max-height: 400px;
    }
  }
  
  @media (min-width: 992px) {
    .button-group {
      grid-template-columns: 1fr 1fr 1fr 1fr;
    }
  }
  
  .control-section {
    background: #2A2A2A;
    border-radius: 8px;
    padding: 12px;
    margin-bottom: 10px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2) inset;
  }
  
  /* 基本控制选项卡内容 */
  .panel-content[v-show="activeTab === 'basic'"] .control-section {
    display: flex;
    flex-wrap: wrap;
    align-items: flex-start;
    gap: 15px;
  }
  
  .panel-content[v-show="activeTab === 'basic'"] .slider-container {
    flex: 1;
    min-width: 250px;
  }
  
  .panel-content[v-show="activeTab === 'basic'"] .button-group {
    flex: 1;
    min-width: 250px;
    margin-top: 0;
  }
  
  /* 背景和场地控制选项卡布局 */
  .panel-content[v-show="activeTab === 'background'"] .control-section,
  .panel-content[v-show="activeTab === 'ground'"] .control-section {
    display: flex;
    flex-wrap: wrap;
    gap: 15px;
  }
  
  /* 相机控制选项卡布局 */
  .panel-content[v-show="activeTab === 'camera'"] .control-section {
    display: flex;
    flex-wrap: wrap;
    gap: 15px;
  }
  
  .panel-content[v-show="activeTab === 'camera'"] .slider-container {
    flex: 1;
    min-width: 250px;
    margin-bottom: 10px;
  }
  
  .panel-content[v-show="activeTab === 'camera'"] .button-group {
    width: 100%;
    display: flex;
    justify-content: space-between;
    margin-top: 10px;
  }
  
  .panel-content h3 {
    width: 100%;
  }
  
  .color-buttons {
    flex: 1;
    min-width: 250px;
  }
  
  .custom-color, .gradient-slider, .object-selector {
    flex: 1;
    min-width: 250px;
  }
  
  .control-section h3 {
    margin: 0 0 10px 0;
    font-size: 16px;
    color: #ccc;
  }
  
  .position-control-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 8px;
    width: 100%;
  }
  
  .position-input {
    width: 70px;
    padding: 4px 8px;
    background: #333;
    border: 1px solid #444;
    border-radius: 4px;
    color: white;
    font-family: monospace;
    text-align: right;
  }
  
  /* 优化移动端显示 */
  @media (max-width: 576px) {
    .position-control-header {
      flex-direction: column;
      align-items: flex-start;
      gap: 5px;
    }
    
    .position-input {
      width: 100%;
    }
  }
  
  .manual-position-set {
    width: 100%;
    background: #2A2A2A;
    border-radius: 6px;
    padding: 12px;
    margin-top: 15px;
  }
  
  .manual-position-set h4 {
    margin: 0 0 10px 0;
    font-size: 15px;
    color: #ccc;
  }
  
  .position-inputs {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 10px;
    margin-bottom: 15px;
    align-items: center;
  }
  
  .position-input-group {
    display: flex;
    align-items: center;
    gap: 8px;
  }
  
  .position-input-group label {
    font-weight: bold;
    color: #ccc;
    min-width: 20px;
  }
  
  .position-input-group .position-input {
    flex: 1;
    min-width: 0;
  }
  
  .preset-positions {
    margin-top: 15px;
  }
  
  .preset-buttons {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 10px;
  }
  
  .preset-btn {
    background: #444;
    color: white;
    border: none;
    border-radius: 4px;
    padding: 5px 10px;
    cursor: pointer;
    transition: all 0.2s;
    font-size: 13px;
  }
  
  .preset-btn:hover {
    background: #555;
  }
  
  /* 移动端适配 */
  @media (max-width: 768px) {
    .position-inputs {
      grid-template-columns: repeat(2, 1fr);
    }
    
    .preset-buttons {
      grid-template-columns: repeat(2, 1fr);
    }
  }
  
  @media (max-width: 576px) {
    .position-inputs {
      grid-template-columns: 1fr;
    }
    
    .preset-buttons {
      grid-template-columns: 1fr 1fr;
    }
  }

  /* 弹窗样式 */
  .dialog-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(0, 0, 0, 0.7);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
    backdrop-filter: blur(5px);
  }
  
  .dialog-content {
    background: linear-gradient(135deg, rgba(43, 88, 118, 0.9) 0%, rgba(78, 67, 118, 0.9) 100%);
    width: 80%;
    max-width: 900px;
    max-height: 80vh;
    border-radius: 15px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
    display: flex;
    flex-direction: column;
    overflow: hidden;
    border: 1px solid rgba(255, 255, 255, 0.1);
  }
  
  .dialog-header {
    padding: 15px 20px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
    background: rgba(0, 0, 0, 0.2);
  }
  
  .dialog-header h2 {
    margin: 0;
    color: white;
    font-size: 18px;
    font-weight: 600;
  }
  
  .dialog-actions {
    display: flex;
    gap: 10px;
  }
  
  .dialog-btn {
    padding: 8px 15px;
    border-radius: 6px;
    border: none;
    color: white;
    cursor: pointer;
    font-size: 14px;
    transition: all 0.2s ease;
  }
  
  .copy-btn {
    background: linear-gradient(90deg, #5f6cef, #c476f1);
  }
  
  .copy-btn:hover {
    background: linear-gradient(90deg, #4a57d9, #b05ae0);
    transform: translateY(-2px);
  }
  
  .close-btn {
    background: rgba(255, 255, 255, 0.2);
  }
  
  .close-btn:hover {
    background: rgba(255, 255, 255, 0.3);
    transform: translateY(-2px);
  }
  
  .dialog-body {
    padding: 20px;
    overflow-y: auto;
    max-height: calc(80vh - 60px);
    color: white;
  }
  
  .dialog-body pre {
    margin: 0;
    white-space: pre-wrap;
    word-wrap: break-word;
    background: rgba(0, 0, 0, 0.2);
    border-radius: 8px;
    padding: 15px;
    overflow-x: auto;
    color: #f0f0f0;
    font-family: 'Consolas', monospace;
    font-size: 14px;
    line-height: 1.5;
  }
  
  /* 响应式适配 */
  @media (max-width: 768px) {
    .dialog-content {
      width: 95%;
      max-height: 90vh;
    }
    
    .dialog-body {
      max-height: calc(90vh - 60px);
    }
  }
  </style> 