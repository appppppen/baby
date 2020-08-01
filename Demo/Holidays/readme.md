### Holidays Demo

```typescript
import Vue from 'vue';
import {
    Vector3, Animation, HemisphericLight, DirectionalLight,
    Scene, AbstractMesh, Engine, ArcRotateCamera, SceneLoader,
    Texture, Sound, Color3, QuarticEase, EasingFunction,
    ParticleHelper, Color4, NoiseProceduralTexture, ParticleSystem,
    NodeMaterial, InputBlock, Tools
} from '@babylonjs/core/Legacy/legacy';
import '@babylonjs/loaders';
import { AdvancedDynamicTexture, Image, Button, Control, TextBlock } from '@babylonjs/gui';
import { ViewModel } from '../ViewModel';
import { Base3DScene } from '../../../../babylon/template/Base3DScene';

import * as snowButton from '../sub_static/snowButton.png';
import * as Dot from '../sub_static/Dot.png';
import * as Happy_Holidays from '../sub_static/Happy_Holidays.mp3';


export class AssembleScene extends Base3DScene {
    viewModel: ViewModel;
    edgesWidth = 6;
    advancedTexture: AdvancedDynamicTexture;
    camera: ArcRotateCamera;
    checked = false;
    sceneChecked = false;
    DialLabel: Image;
    snow: ParticleSystem;
    cameraPos: Vector3;

    snowHeight: InputBlock;
    gTransition: InputBlock;
    tTransition: InputBlock;

    groundAccumulation: Animation;
    groundTransition: Animation;
    treeTransition: Animation;
    menuMotion: Animation;
    titleMotion: Animation;

    // Set up menu button
    button: Button;
    buttonVisible = true;
    holidayLabel: TextBlock;

    music: Sound;
    snowing = false;
    groundMesh: AbstractMesh;
    treeMesh: AbstractMesh;
    playing = false;

    groundNodeMat: NodeMaterial;
    treeNodeMat: NodeMaterial;

    constructor(vm: Vue) {
        super();
        this.viewModel = vm as ViewModel;
        this.init();
    }

    createCamera(canvas: HTMLCanvasElement, scene: Scene) {
        this.camera = new ArcRotateCamera('ArcRotateCamera', Tools.ToRadians(-120), Tools.ToRadians(85), 12, new Vector3(0, 1.5, 0), scene);
        this.camera.lowerRadiusLimit = 8;
        this.camera.upperRadiusLimit = 20;
        this.camera.minZ = 1.0;
        scene.activeCameras.push(this.camera);
        this.camera.attachControl(canvas, true);
    }

    createLight(scene: Scene) {
        const dirLight1 = new DirectionalLight('dirLight1', new Vector3(0, 0, 0), scene);
        dirLight1.direction = new Vector3(1, -1, 0.5);
        dirLight1.intensity = 0.95;
        dirLight1.diffuse = new Color3(0.8431372549019608, 0.8431372549019608, 0.8431372549019608);
        dirLight1.specular = new Color3(0.7568627450980392, 0.7568627450980392, 0.7568627450980392);
        const hemLight = new HemisphericLight('hemLight', new Vector3(0, 1, 0), scene);
        hemLight.intensity = 0.5;
    }
    /** 
     * 创建场景
     */
    createScene(engine: Engine): Scene {
        const canvas = engine.getRenderingCanvas();
        const scene = new Scene(engine);
        scene.clearColor = new Color4(0.1, 0.1, 0.1, 1);
        this.advancedTexture = AdvancedDynamicTexture.CreateFullscreenUI('UI');
        const promises = [];
        this.createCamera(canvas, scene);
        this.music = new Sound('Music', Happy_Holidays, scene, null, { loop: true, autoplay: false });
        this.createLight(scene);

        this.groundNodeMat = new NodeMaterial('groundNodeMat', scene, { emitComments: false });
        this.treeNodeMat = new NodeMaterial('treeNodeMat', scene, { emitComments: false });
        this.createAnima();

        promises.push(
            SceneLoader.AppendAsync('https://appppppen.github.io/art/Holidays/treeScene.glb'));
        promises.push(
            this.groundNodeMat.loadAsync('https://appppppen.github.io/art/Holidays/groundNodeShader.json'));
        promises.push(
            this.treeNodeMat.loadAsync('https://appppppen.github.io/art/Holidays/treeNodeShader.json'));

        Promise.all(promises).then(() => {
            this.importMeshSuccess(scene);
        });

        return scene;
    }

    createMat(scene: Scene) {
        this.snowHeight = this.groundNodeMat.getBlockByName('snowHeight') as InputBlock;
        this.snowHeight.value = 0.0;
        this.gTransition = this.groundNodeMat.getBlockByName('transition') as InputBlock;
        this.gTransition.value = -0.1;
        this.tTransition = this.treeNodeMat.getBlockByName('transition') as InputBlock;
        this.tTransition.value = -0.1;
    }

    /**
     * 导入模型
     * @param meshes 
     */
    importMeshSuccess(scene: Scene) {
        const groundMesh = scene.getMeshByName('ground_high');
        const treeMesh = scene.getMeshByName('tree_low');

        //Build and assign node materials
        this.groundNodeMat.build(true);
        this.groundNodeMat.needDepthPrePass = true;
        groundMesh.material = this.groundNodeMat;

        this.treeNodeMat.build(true);
        this.treeNodeMat.needDepthPrePass = true;
        treeMesh.material = this.treeNodeMat;

        this.createMat(scene);
        this.snow = this.createSnow(scene);
        this.button = this.createBtn();
        this.advancedTexture.addControl(this.button);
        this.holidayLabel = this.createholidayLabel();
        this.advancedTexture.addControl(this.holidayLabel);

        const buttonHoverColor = new Color3(64 / 255, 145 / 255, 172 / 255);
        const buttonClickColor = new Color3(249 / 255, 244 / 255, 120 / 255);
        const buttonDefaultColor = new Color3(1.0, 1.0, 1.0);

        this.button.pointerEnterAnimation = () => {
            if (this.buttonVisible) { this.button.color = buttonHoverColor.toHexString(); }
        };

        this.button.pointerOutAnimation = () => {
            if (this.buttonVisible) { this.button.color = buttonDefaultColor.toHexString(); }
        };

        this.button.pointerDownAnimation = () => {
            if (this.buttonVisible) { this.button.color = buttonClickColor.toHexString(); }
        };

        this.button.pointerUpAnimation = () => {
            if (this.buttonVisible) {
                this.button.color = buttonDefaultColor.toHexString();
                this.buttonVisible = false;
                if (!this.music.isPlaying) {
                    this.music.play();
                }
                this.playing = !this.playing;
                this.fadeMusic(this.playing);
                this.updateGUI();
                this.toggleSnow();
            }
        };
    }

    createBtn(): Button {
        const button = Button.CreateImageWithCenterTextButton('snowButton', 'Start Snowing', snowButton);
        button.height = '135px';
        button.width = '367px';
        button.horizontalAlignment = Control.HORIZONTAL_ALIGNMENT_CENTER;
        button.verticalAlignment = Control.VERTICAL_ALIGNMENT_BOTTOM;
        const buttonDefaultColor = new Color3(1.0, 1.0, 1.0);
        button.thickness = 0;
        button.textBlock.fontSize = '40px';
        button.textBlock.fontFamily = 'viktor-script';
        button.textBlock.textHorizontalAlignment = Control.HORIZONTAL_ALIGNMENT_LEFT;
        button.color = buttonDefaultColor.toHexString();
        button.textBlock.paddingTop = '45px';
        button.textBlock.paddingLeft = '110px';
        return button;
    }

    createholidayLabel(): TextBlock {
        const holidayLabel = new TextBlock('happyHolidays');
        holidayLabel.text = 'Happy Holidays';
        holidayLabel.fontSize = '100px';
        holidayLabel.fontFamily = 'viktor-script';
        holidayLabel.color = '#e7d55b';
        holidayLabel.height = '200px';
        holidayLabel.top = '-200px';
        holidayLabel.textHorizontalAlignment = Control.HORIZONTAL_ALIGNMENT_CENTER;
        holidayLabel.verticalAlignment = Control.VERTICAL_ALIGNMENT_TOP;
        return holidayLabel;
    }

    createSnow(scene: Scene): ParticleSystem {
        const particleSource = new AbstractMesh('snowSource', scene);
        particleSource.position = new Vector3(0, 8, 0);
        const snow = ParticleHelper.CreateDefault(particleSource, 4000);
        snow.particleTexture = new Texture(Dot, scene);
        snow.createBoxEmitter(new Vector3(0, -1, 0), new Vector3(0, -1, 0), new Vector3(2.5, 0, 2.5), new Vector3(-2.5, 0, -2.5));
        snow.minLifeTime = 7;
        snow.maxLifeTime = 7;
        snow.emitRate = 300;
        snow.minEmitPower = 0.25;
        snow.maxEmitPower = 0.5;
        snow.gravity = new Vector3(0, -0.25, 0);
        snow.minSize = 0.02;
        snow.maxSize = 0.03;
        snow.addColorGradient(0, new Color4(0.95, 0.95, 0.95, 1.0), new Color4(0.8, 0.8, 0.8, 1.0));
        snow.addColorGradient(0.8, new Color4(0.95, 0.95, 0.95, 0.71), new Color4(0.8, 0.8, 0.8, 0.71));
        snow.addColorGradient(1.0, new Color4(0.95, 0.95, 0.95, 0.0), new Color4(0.8, 0.8, 0.8, 0.0));

        // snowfall path noise
        const noiseTexture = new NoiseProceduralTexture('perlin', 16, scene);
        noiseTexture.animationSpeedFactor = 3;
        noiseTexture.persistence = 0.4;
        noiseTexture.brightness = 0.5;
        noiseTexture.octaves = 3;
        snow.noiseTexture = noiseTexture;
        snow.noiseStrength = new Vector3(0.75, 0, 0.5);
        // snow blend mode
        (snow as ParticleSystem).forceDepthWrite = true;
        snow.blendMode = ParticleSystem.BLENDMODE_STANDARD;
        return (snow as ParticleSystem);
    }

    // toggle snow
    toggleSnow() {
        this.updateSnow(this.scene);
        if (this.snowing) {
            this.snow.stop();
            this.snowing = false;
        } else {
            this.snow.start();
            this.snowing = true;
        }
    }

    createAnima() {
        this.groundAccumulation = new Animation('groundAccumulation', 'value', 60,
            Animation.ANIMATIONTYPE_FLOAT, Animation.ANIMATIONLOOPMODE_CONSTANT);
        this.groundTransition = new Animation('groundTransition', 'value', 60,
            Animation.ANIMATIONTYPE_FLOAT, Animation.ANIMATIONLOOPMODE_CONSTANT);
        this.treeTransition = new Animation('treeTransition', 'value', 60,
            Animation.ANIMATIONTYPE_FLOAT, Animation.ANIMATIONLOOPMODE_CONSTANT);
        this.menuMotion = new Animation('menuMotion', 'top', 60,
            Animation.ANIMATIONTYPE_FLOAT, Animation.ANIMATIONLOOPMODE_CONSTANT);
        this.titleMotion = new Animation('titleMotion', 'top', 60,
            Animation.ANIMATIONTYPE_FLOAT, Animation.ANIMATIONLOOPMODE_CONSTANT);

        const easingFunction = new QuarticEase();
        easingFunction.setEasingMode(EasingFunction.EASINGMODE_EASEINOUT);
        this.groundAccumulation.setEasingFunction(easingFunction);
        this.groundTransition.setEasingFunction(easingFunction);
        this.treeTransition.setEasingFunction(easingFunction);
        this.menuMotion.setEasingFunction(easingFunction);
        this.titleMotion.setEasingFunction(easingFunction);
    }

    gTransitionK() {
        const transitionKeys = [];
        transitionKeys.length = 0;
        transitionKeys.push({ frame: 0, value: 1.1 });
        transitionKeys.push({ frame: 300, value: -0.1 });
        this.groundTransition.setKeys(transitionKeys);
        this.scene.beginDirectAnimation(this.gTransition, [this.groundTransition],
            0, transitionKeys[transitionKeys.length - 1].frame, false, 1,
            () => {
                this.updateText(false);
                this.updateGUI();
            });
    }

    updateSnow(scene: Scene) {
        const transitionKeys = [];
        const treeTransitionKeys = [];
        const accumulationKeys = [];
        if (this.snowing) {
            accumulationKeys.length = 0;
            accumulationKeys.push({ frame: 0, value: 1.0 });
            accumulationKeys.push({ frame: 300, value: 1.0 });
            accumulationKeys.push({ frame: 700, value: 0.0 });
            this.groundAccumulation.setKeys(accumulationKeys);

            treeTransitionKeys.length = 0;
            treeTransitionKeys.push({ frame: 0, value: 0.6 });
            treeTransitionKeys.push({ frame: 200, value: 0.6 });
            treeTransitionKeys.push({ frame: 1300, value: -0.1 });
            this.treeTransition.setKeys(treeTransitionKeys);

            scene.beginDirectAnimation(this.tTransition, [this.treeTransition], 0,
                treeTransitionKeys[treeTransitionKeys.length - 1].frame, false, 1);
            scene.beginDirectAnimation(this.snowHeight, [this.groundAccumulation], 0,
                accumulationKeys[accumulationKeys.length - 1].frame, false, 1, () => { this.gTransitionK(); });
        } else {
            transitionKeys.length = 0;
            transitionKeys.push({ frame: 0, value: -0.1 });
            transitionKeys.push({ frame: 300, value: -0.1 });
            transitionKeys.push({ frame: 700, value: 1.1 });
            this.groundTransition.setKeys(transitionKeys);

            treeTransitionKeys.length = 0;
            treeTransitionKeys.push({ frame: 0, value: -0.1 });
            treeTransitionKeys.push({ frame: 100, value: -0.1 });
            treeTransitionKeys.push({ frame: 1300, value: 0.6 });
            this.treeTransition.setKeys(treeTransitionKeys);

            this.animateTitle();
            scene.beginDirectAnimation(this.tTransition, [this.treeTransition], 0,
                treeTransitionKeys[treeTransitionKeys.length - 1].frame, false, 1);
            scene.beginDirectAnimation(this.gTransition, [this.groundTransition], 0,
                transitionKeys[transitionKeys.length - 1].frame, false, 1, () => { this.accumulation(); });
        }
    }

    accumulation() {
        const accumulationKeys = [];
        accumulationKeys.length = 0;
        accumulationKeys.push({ frame: 0, value: 0.0 });
        accumulationKeys.push({ frame: 200, value: 1.0 });
        this.groundAccumulation.setKeys(accumulationKeys);
        this.scene.beginDirectAnimation(this.snowHeight, [this.groundAccumulation], 0,
            accumulationKeys[accumulationKeys.length - 1].frame, false, 1,
            () => {
                this.updateText(true);
                this.updateGUI();
            });
    }

    updateText(isGo: boolean) {
        if (isGo) {
            this.button.textBlock.text = 'Stop Snowing';
            this.buttonVisible = true;
        } else {
            this.button.textBlock.text = 'Start Snowing';
            this.buttonVisible = true;
        }
    }

    updateGUI() {
        const menuMotionKeys = [];
        if (this.buttonVisible) {
            menuMotionKeys.length = 0;
            menuMotionKeys.push({ frame: 0, value: 200 });
            menuMotionKeys.push({ frame: 60, value: 0 });
            this.menuMotion.setKeys(menuMotionKeys);
        } else {
            menuMotionKeys.length = 0;
            menuMotionKeys.push({ frame: 0, value: 0 });
            menuMotionKeys.push({ frame: 60, value: 200 });
            this.menuMotion.setKeys(menuMotionKeys);
        }
        this.scene.beginDirectAnimation(this.button, [this.menuMotion], 0, menuMotionKeys[menuMotionKeys.length - 1].frame,
            false, 1);
    }

    animateTitle() {
        const titleMotionKeys = [];
        if (titleMotionKeys.length === 0) {
            this.holidayLabel.text = 'Happy Holidays';
            titleMotionKeys.push({ frame: 0, value: -200 });
            titleMotionKeys.push({ frame: 200, value: -200 });
            titleMotionKeys.push({ frame: 300, value: 50 });
            titleMotionKeys.push({ frame: 900, value: 50 });
            titleMotionKeys.push({ frame: 1000, value: -200 });
            this.titleMotion.setKeys(titleMotionKeys);
        }
        this.scene.beginDirectAnimation(this.holidayLabel, [this.titleMotion], 0,
            titleMotionKeys[titleMotionKeys.length - 1].frame, false, 1);
    }

    fadeMusic(fadeIn: boolean) {
        const musicInterval = 0.003;
        if (fadeIn) {
            const i = setInterval(() => {
                if (this.music.getVolume() < 1) {
                    this.music.setVolume(this.music.getVolume() + musicInterval);
                } else {
                    this.music.setVolume(1);
                    clearInterval(i);
                }
            }, 10);
        } else {
            const i = setInterval(() => {
                if (this.music.getVolume() > 0) {
                    this.music.setVolume(this.music.getVolume() - musicInterval);
                } else {
                    this.music.setVolume(0);
                    clearInterval(i);
                }
            }, 10);
        }
    }

    /**
     * 重置按钮按下
     */
    reset(): void {

    }
}

```
