### Use GizmoManager

```typescript
import {
    Vector3, Engine, Scene, Animation, Mesh, Color3, PointLight, PBRMaterial,
    ImageProcessingConfiguration, QuarticEase, EasingFunction,
    CubeTexture, AbstractMesh, SceneLoader, ArcRotateCamera, Quaternion, GizmoManager, Matrix
} from '@babylonjs/core/Legacy/legacy';
import '@babylonjs/loaders';
import { BaseScene } from '../../../../babylon/template/Base/BaseScene';
import { AdvancedDynamicTexture, Control, Button } from '@babylonjs/gui';

export class AssembleScene extends BaseScene {

    highlightGizmo(gizmo) {
        const color = gizmo._rootMesh.getChildMeshes()[0].material.emissiveColor;
        color.addToRef(new Color3(0.3, 0.3, 0.3), color);
    }

    unhighlightGizmo(gizmo) {
        const color = gizmo._rootMesh.getChildMeshes()[0].material.emissiveColor;
        color.addToRef(new Color3(-0.3, -0.3, -0.3), color);
    }

    /* 创建场景*/
    createScene(engine: Engine): Scene {
        const canvas = engine.getRenderingCanvas();
        const scene = new Scene(engine);
        scene.clearColor.set(0.2, 0.2, 0.2, 1);

        const camera = new ArcRotateCamera('cam', 1.2, Math.PI / 2.5, 3, Vector3.Zero(), scene);
        camera.wheelDeltaPercentage = 0.01;
        camera.attachControl(canvas, true);

        const cameraParent = new AbstractMesh('cameraParent', scene);
        camera.parent = cameraParent;
        cameraParent.position = new Vector3(0, 5, 0);

        // To prevent limitations of lights per mesh
        engine.disableUniformBuffers = true;

        // Material
        const groundMat = new PBRMaterial('groundMat', scene);
        groundMat.albedoColor = new Color3(93 / 255, 93 / 255, 93 / 255);
        groundMat.metallic = 0;
        groundMat.roughness = 0.4;
        groundMat.environmentIntensity = 0;
        groundMat.maxSimultaneousLights = 16;

        // Ground
        const ground = Mesh.CreatePlane('ground', 500.0, scene);
        ground.position = new Vector3(0, -1, 0);
        ground.rotation = new Vector3(Math.PI / 2, 0, 0);
        ground.material = groundMat;

        // Lights
        const light = new PointLight('pointLight', new Vector3(0, 2.5, 0), scene);
        light.intensity = 0;
        light.includedOnlyMeshes.push(ground);

        // Environment
        const hdrTexture = CubeTexture.CreateFromPrefilteredData(
            'https://raw.githubusercontent.com/PatrickRyanMS/BabylonJStextures/master/DDS/Studio_Softbox_2Umbrellas_cube_specular.dds', scene);
        hdrTexture.name = 'envTex';
        hdrTexture.gammaSpace = false;
        scene.environmentTexture = hdrTexture;
        // scene._environmentBRDFTexture = new Texture(
        //     'https://assets.babylonjs.com/environments/fullFloatEnvironmentBrdf.dds', scene, true, true);
        // scene._environmentBRDFTexture.wrapU = Texture.CLAMP_ADDRESSMODE;
        // scene._environmentBRDFTexture.wrapV = Texture.CLAMP_ADDRESSMODE;

        // Set up new rendering pipeline
        // const pipeline = new DefaultRenderingPipeline('default', true, scene);

        // Anti-aliasing
        // pipeline.samples = 4;
        // pipeline.grainEnabled = true;
        // pipeline.grain.intensity = 1;

        // Tone mapping
        scene.imageProcessingConfiguration.toneMappingEnabled = true;
        scene.imageProcessingConfiguration.toneMappingType = ImageProcessingConfiguration.TONEMAPPING_ACES;
        scene.imageProcessingConfiguration.exposure = 3;

        // Bloom
        // pipeline.bloomEnabled = true;
        // pipeline.bloomThreshold = 1;
        // pipeline.bloomWeight = 0.15;
        // pipeline.bloomKernel = 64;
        // pipeline.bloomScale = 0.4;

        //Animation
        const lightOn = new Animation('lightOn', 'intensity', 60, Animation.ANIMATIONTYPE_FLOAT, Animation.ANIMATIONLOOPMODE_CONSTANT);
        const lightOff = new Animation('lightOff', 'intensity', 60, Animation.ANIMATIONTYPE_FLOAT, Animation.ANIMATIONLOOPMODE_CONSTANT);
        const meshLightOn =
            new Animation('meshLightOn', 'intensity', 60, Animation.ANIMATIONTYPE_FLOAT, Animation.ANIMATIONLOOPMODE_CONSTANT);
        const meshLightOff =
            new Animation('meshLightOff', 'intensity', 60, Animation.ANIMATIONTYPE_FLOAT, Animation.ANIMATIONLOOPMODE_CONSTANT);
        const hydrate = new Animation('hydrate', 'position.y', 60, Animation.ANIMATIONTYPE_FLOAT, Animation.ANIMATIONLOOPMODE_CONSTANT);
        const dehydrate = new Animation('dehydrate', 'position.y', 60, Animation.ANIMATIONTYPE_FLOAT, Animation.ANIMATIONLOOPMODE_CONSTANT);
        const hydrateCamera =
            new Animation('hydrateCamera', 'position.y', 60, Animation.ANIMATIONTYPE_FLOAT, Animation.ANIMATIONLOOPMODE_CONSTANT);
        const pushCamera = new
            Animation('pushCamera', 'position.y', 60, Animation.ANIMATIONTYPE_FLOAT, Animation.ANIMATIONLOOPMODE_CONSTANT);
        const dehydrateCamera =
            new Animation('dehydrateCamera', 'position.y', 60, Animation.ANIMATIONTYPE_FLOAT, Animation.ANIMATIONLOOPMODE_CONSTANT);
        const meshMove = new Animation('meshMove', 'position', 60, Animation.ANIMATIONTYPE_VECTOR3, Animation.ANIMATIONLOOPMODE_RELATIVE);
        const meshRotate =
            new Animation('meshMove', 'rotationQuaternion', 60, Animation.ANIMATIONTYPE_QUATERNION, Animation.ANIMATIONLOOPMODE_RELATIVE);
        const meshScale = new Animation('meshMove', 'scaling', 60, Animation.ANIMATIONTYPE_VECTOR3, Animation.ANIMATIONLOOPMODE_RELATIVE);

        // Animation keys and values
        const keysLightOn = [];
        keysLightOn.push({ frame: 0, value: 0 });
        keysLightOn.push({ frame: 40, value: 6 });

        const keysLightOff = [];
        keysLightOff.push({ frame: 0, value: 6 });
        keysLightOff.push({ frame: 60, value: 0 });

        const keysMeshLightOn = [];
        keysMeshLightOn.push({ frame: 0, value: 0 });
        keysMeshLightOn.push({ frame: 40, value: 1.5 });

        const keysMeshLightOff = [];
        keysMeshLightOff.push({ frame: 0, value: 1.5 });
        keysMeshLightOff.push({ frame: 60, value: 0 });

        const keysHydrate = [];
        keysHydrate.push({ frame: 0, value: 5 });
        keysHydrate.push({ frame: 60, value: 0.7 });

        const keysDehydrate = [];
        keysDehydrate.push({ frame: 0, value: 0.7 });
        keysDehydrate.push({ frame: 60, value: 5 });

        const keysHydrateCamera = [];
        keysHydrateCamera.push({ frame: 0, value: 5 });
        keysHydrateCamera.push({ frame: 60, value: 0 });

        const keysPushCamera = [];
        keysPushCamera.push({ frame: 0, value: 0 });
        keysPushCamera.push({ frame: 19, value: 0 });
        keysPushCamera.push({ frame: 60, value: 1.3 });

        const keysDehydrateCamera = [];
        keysDehydrateCamera.push({ frame: 0, value: 0 });
        keysDehydrateCamera.push({ frame: 60, value: 5 });

        const keysMeshMove = [];
        keysMeshMove.push({ frame: 0, value: new Vector3(0, 0, 0) });
        keysMeshMove.push({ frame: 60, value: new Vector3(-0.3616832944873265, 0, 0.3431647967513791) });

        const keysMeshRotate = [];
        keysMeshRotate.push({ frame: 0, value: new Quaternion(0, 0, 0, 1) });
        keysMeshRotate.push({ frame: 60, value: new Quaternion(0, -0.9264498872012958, 0, 0.37641812722649837) });

        const keysMeshScale = [];
        keysMeshScale.push({ frame: 0, value: new Vector3(1, 1, 1) });
        keysMeshScale.push({ frame: 60, value: new Vector3(1.2316533224592008, 1.2316533224592008, 1.2316533224592008) });

        // Adding keys to the animation
        lightOn.setKeys(keysLightOn);
        lightOff.setKeys(keysLightOff);
        meshLightOn.setKeys(keysMeshLightOn);
        meshLightOff.setKeys(keysMeshLightOff);
        hydrate.setKeys(keysHydrate);
        dehydrate.setKeys(keysDehydrate);
        hydrateCamera.setKeys(keysHydrateCamera);
        pushCamera.setKeys(keysPushCamera);
        dehydrateCamera.setKeys(keysDehydrateCamera);
        meshMove.setKeys(keysMeshMove);
        meshRotate.setKeys(keysMeshRotate);
        meshScale.setKeys(keysMeshScale);

        const easingFunction = new QuarticEase();

        // For each easing function, you can choose between EASEIN (default), EASEOUT, EASEINOUT
        easingFunction.setEasingMode(EasingFunction.EASINGMODE_EASEINOUT);

        // Adding the easing function to the animation
        lightOn.setEasingFunction(easingFunction);
        lightOff.setEasingFunction(easingFunction);
        meshLightOn.setEasingFunction(easingFunction);
        meshLightOff.setEasingFunction(easingFunction);
        hydrate.setEasingFunction(easingFunction);
        dehydrate.setEasingFunction(easingFunction);
        hydrateCamera.setEasingFunction(easingFunction);
        pushCamera.setEasingFunction(easingFunction);
        dehydrateCamera.setEasingFunction(easingFunction);
        meshMove.setEasingFunction(easingFunction);
        meshRotate.setEasingFunction(easingFunction);
        meshScale.setEasingFunction(easingFunction);

        // Abstract meshes for animation
        const controllerAbstract = new AbstractMesh('controllerAbstract', scene);
        const lightAbstract = new AbstractMesh('lightAbstract', scene);

        function AddLight(parentMesh) {
            const buttonLight = new PointLight('meshLight',
                new Vector3(parentMesh.position.x, parentMesh.position.y - 0.3, parentMesh.position.z), scene);
            buttonLight.parent = parentMesh;
            buttonLight.includedOnlyMeshes.push(ground);
            buttonLight.intensity = 0.0;
            return buttonLight;
        }

        const meshLight = AddLight(lightAbstract);
        console.log('Mesh Light: ' + meshLight);

        // Create and initialize gizmo
        const gizmoManager = new GizmoManager(scene);
        gizmoManager.usePointerToAttachGizmos = false;
        gizmoManager.positionGizmoEnabled = true;
        gizmoManager.rotationGizmoEnabled = true;
        gizmoManager.scaleGizmoEnabled = true;
        gizmoManager.positionGizmoEnabled = false;
        gizmoManager.rotationGizmoEnabled = false;
        gizmoManager.scaleGizmoEnabled = false;

        // Load glTF model and assign to Gizmo and add grounding light
        // SceneLoader.ImportMesh('', 'meshes/', 'PBR - Metallic Roughness.glb', scene, function (newMeshes) {
        SceneLoader.ImportMesh('',
            'https://raw.githubusercontent.com/PatrickRyanMS/SampleModels/master/XboxEliteController/glb/',
            'PBR - Metallic Roughness.glb', scene, (newMeshes) => {
                const gltfMesh = newMeshes[0];
                gltfMesh.rotate(new Vector3(0, 1, 0), Math.PI);
                gltfMesh.rotate(new Vector3(1, 0, 0), Math.PI / 3);
                gltfMesh.position = new Vector3(0, 0, -0.4);
                gltfMesh.scaling = new Vector3(-0.1, 0.1, 0.1);
                gltfMesh.parent = controllerAbstract;
                gizmoManager.attachableMeshes = [controllerAbstract];
                gizmoManager.attachToMesh(controllerAbstract);
            });

        let i = 0;


        function SelectGizmo(event) {
            if (event.keyCode === 81 || event.keyCode === 87 || event.keyCode === 69 || event.keyCode === 82 || event.keyCode === 75) {
                // Switch gizmo type
                gizmoManager.positionGizmoEnabled = false;
                gizmoManager.rotationGizmoEnabled = false;
                gizmoManager.scaleGizmoEnabled = false;

                if (event.keyCode === 81) {
                    console.log('No gizmo');
                }

                if (event.keyCode === 87) {
                    gizmoManager.positionGizmoEnabled = true;
                    gizmoManager.gizmos.positionGizmo.updateGizmoRotationToMatchAttachedMesh = true;
                    console.log('Position gizmo');
                }

                if (event.keyCode === 69) {
                    gizmoManager.rotationGizmoEnabled = true;
                    gizmoManager.gizmos.rotationGizmo.updateGizmoRotationToMatchAttachedMesh = true;
                    console.log('Rotation gizmo');
                }

                if (event.keyCode === 82) {
                    gizmoManager.scaleGizmoEnabled = true;
                    console.log('Scale gizmo');
                }

                if (event.keyCode === 75) {
                    controllerAbstract.position = new Vector3(0, 0, 0);
                    controllerAbstract.rotationQuaternion = new Quaternion(0, 0, 0, 1);
                    controllerAbstract.scaling = new Vector3(1, 1, 1);
                    console.log('Reset gizmo');
                }
            }
        }

        function CameraMove(event) {
            if (event.keyCode === 32) {
                // Space bar to animate the camera to aim at controller and turn on lights. Reversable with space bar. 

                if (i === 0) {
                    // Add animation to light
                    scene.beginDirectAnimation(meshLight, [meshLightOn], 0, 40, false, 1);
                    scene.beginDirectAnimation(light, [lightOn], 0, 40, false, 1);
                    scene.beginDirectAnimation(cameraParent, [hydrateCamera], 0, 60, false, 1);
                    i++;
                } else {
                    // Add animation to light
                    scene.beginDirectAnimation(cameraParent, [dehydrateCamera], 0, 60, false, 1);
                    scene.beginDirectAnimation(meshLight, [meshLightOff], 0, 60, false, 1);
                    scene.beginDirectAnimation(light, [lightOff], 0, 60, false, 1);
                    i = 0;
                }
            }
        }

        function AutomateTransform(event) {
            if (event.keyCode === 49) {// 1 key to automatically rotate controller
                this.highlightGizmo(gizmoManager.gizmos.rotationGizmo.yGizmo);
                scene.beginDirectAnimation(controllerAbstract, [meshRotate], 0, 60, false, 1,
                    function () { this.unhighlightGizmo(gizmoManager.gizmos.rotationGizmo.yGizmo); });
            }
            if (event.keyCode === 50) {// 2 key to automatically scale controller
                this.highlightGizmo(gizmoManager.gizmos.scaleGizmo.uniformScaleGizmo);
                scene.beginDirectAnimation(controllerAbstract, [meshScale], 0, 60, false, 1,
                    function () { this.unhighlightGizmo(gizmoManager.gizmos.scaleGizmo.uniformScaleGizmo); });
            }
            if (event.keyCode === 51) { // 3 key to automatically translate controller
                this.highlightGizmo(gizmoManager.gizmos.positionGizmo.xGizmo);
                scene.beginDirectAnimation(controllerAbstract, [meshMove], 0, 60, false, 1,
                    function () { this.unhighlightGizmo(gizmoManager.gizmos.positionGizmo.xGizmo); });
            }
            if (event.keyCode === 52) { // 4 key to reverse rotate controller
                this.highlightGizmo(gizmoManager.gizmos.rotationGizmo.yGizmo);
                scene.beginDirectAnimation(controllerAbstract, [meshRotate], 60, 0, false, -1,
                    function () { this.unhighlightGizmo(gizmoManager.gizmos.rotationGizmo.yGizmo); });
            }
            if (event.keyCode === 53) {// 5 key to reverse scale controller
                this.highlightGizmo(gizmoManager.gizmos.scaleGizmo.uniformScaleGizmo);
                scene.beginDirectAnimation(controllerAbstract, [meshScale], 60, 0, false, -1,
                    function () { this.unhighlightGizmo(gizmoManager.gizmos.scaleGizmo.uniformScaleGizmo); });
            }
            if (event.keyCode === 54) {// 6 key to reverse translate controller
                this.highlightGizmo(gizmoManager.gizmos.positionGizmo.xGizmo);
                scene.beginDirectAnimation(controllerAbstract, [meshMove], 60, 0, false, -1,
                    function () { this.unhighlightGizmo(gizmoManager.gizmos.positionGizmo.xGizmo); });
            }
        }

        // Inspector
        let showInspector = false;
        function ShowInspector(event) {
            if (event.keyCode === 78) { // N key to show and hide inspector
                if (!showInspector) {
                    scene.debugLayer.show();
                    showInspector = !showInspector;
                } else {
                    scene.debugLayer.hide();
                    showInspector = !showInspector;
                }
            }
        }
        let envRotation = 7.58;

        function RotateEnvironment(event) {
            if (event.keyCode === 221) { // [ key to rotate environment to the left
                envRotation += 0.01;
            }
            if (event.keyCode === 219) {// ] key to to rotate environment to the right
                envRotation -= 0.01;
            }
            if (event.keyCode === 220) {// \ key to print radian value to console for environment rotation
                console.log('Environment Rotation: ' + envRotation);
                console.log('Position: ' + new Vector3(controllerAbstract.position.x,
                    controllerAbstract.position.y, controllerAbstract.position.z));
                console.log('Rotation: ' + controllerAbstract.rotationQuaternion);
                console.log('Scale: ' + controllerAbstract.scaling);
            }
        }
        scene.registerBeforeRender(function () {
            // Set rotation of environment
            hdrTexture.setReflectionTextureMatrix(Matrix.RotationY(envRotation));
            // Position light with mesh
            lightAbstract.position =
                new Vector3(controllerAbstract.position.x, controllerAbstract.position.y, controllerAbstract.position.z);
        });

        scene.beginDirectAnimation(meshLight, [meshLightOn], 0, 40, false, 1);
        scene.beginDirectAnimation(light, [lightOn], 0, 40, false, 1);
        scene.beginDirectAnimation(cameraParent, [hydrateCamera], 0, 60, false, 1);

        const advancedTexture = AdvancedDynamicTexture.CreateFullscreenUI('UI');

        const rotate = Button.CreateSimpleButton('rotate', 'Rotate');
        rotate.width = '100px';
        rotate.height = '35px';
        rotate.color = 'white';
        rotate.background = 'black';
        rotate.horizontalAlignment = Control.HORIZONTAL_ALIGNMENT_RIGHT;
        rotate.paddingRight = '30px';
        rotate.top = '-40px';
        rotate.onPointerDownObservable.add(function () {
            SelectGizmo2('rotate');
        });
        advancedTexture.addControl(rotate);

        const scale = Button.CreateSimpleButton('scale', 'Scale');
        scale.width = '100px';
        scale.height = '35px';
        scale.color = 'white';
        scale.background = 'black';
        scale.horizontalAlignment = Control.HORIZONTAL_ALIGNMENT_RIGHT;
        scale.paddingRight = '30px';
        scale.top = '0px';
        scale.onPointerDownObservable.add(function () {
            SelectGizmo2('scale');
        });
        advancedTexture.addControl(scale);

        const move = Button.CreateSimpleButton('move', 'Move');
        move.width = '100px';
        move.height = '35px';
        move.color = 'white';
        move.background = 'black';
        move.paddingRight = '30px';
        move.top = '40px';
        move.horizontalAlignment = Control.HORIZONTAL_ALIGNMENT_RIGHT;
        move.onPointerDownObservable.add(function () {
            SelectGizmo2('move');
        });
        advancedTexture.addControl(move);

        const reset = Button.CreateSimpleButton('reset', 'Reset');
        reset.width = '100px';
        reset.height = '35px';
        reset.color = 'white';
        reset.background = 'black';
        reset.horizontalAlignment = Control.HORIZONTAL_ALIGNMENT_RIGHT;
        reset.paddingRight = '30px';
        reset.top = '80px';
        reset.onPointerDownObservable.add(function () {
            SelectGizmo2('reset');
        });
        advancedTexture.addControl(reset);

        function SelectGizmo2(button) {
            // Switch gizmo type
            gizmoManager.positionGizmoEnabled = false;
            gizmoManager.rotationGizmoEnabled = false;
            gizmoManager.scaleGizmoEnabled = false;
            if (button === 'move') {
                gizmoManager.positionGizmoEnabled = true;
                gizmoManager.gizmos.positionGizmo.updateGizmoRotationToMatchAttachedMesh = true;
                console.log('Position gizmo');
            }

            if (button === 'rotate') {
                gizmoManager.rotationGizmoEnabled = true;
                gizmoManager.gizmos.rotationGizmo.updateGizmoRotationToMatchAttachedMesh = true;
                console.log('Rotation gizmo');
            }

            if (button === 'scale') {
                gizmoManager.scaleGizmoEnabled = true;
                console.log('Scale gizmo');
            }

            if (button === 'reset') {
                controllerAbstract.position = new Vector3(0, 0, 0);
                controllerAbstract.rotationQuaternion = new Quaternion(0, 0, 0, 1);
                controllerAbstract.scaling = new Vector3(1, 1, 1);
                console.log('Reset gizmo');
            }

        }

        engine.displayLoadingUI();
        scene.executeWhenReady(function () {
            engine.hideLoadingUI();
        });
        return scene;
    }

}

```
