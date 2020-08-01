### Use pipeline.depthOfField

```typescript
import {
    ArcRotateCamera, Scene, FreeCamera, Vector3, Engine, PBRMetallicRoughnessMaterial,
    HemisphericLight, Color3, Mesh, CubeTexture, DefaultRenderingPipeline, DepthOfFieldEffectBlurLevel
} from '@babylonjs/core/Legacy/legacy';
import { BaseScene } from '../../../../babylon/template/Base/BaseScene';
import * as environment from '../sub_static/image/environment.dds';
import { AdvancedDynamicTexture, StackPanel, Control, TextBlock, Slider } from '@babylonjs/gui';

export class AssembleScene extends BaseScene {

    /* 创建相机*/
    createCamera(scene: Scene) {
        const camera = new FreeCamera('camera1', new Vector3(0, 0.3, -0.7), scene);
        camera.speed = 0.01;
        camera.minZ = 0.001;
        scene.activeCameras.push(camera);
        camera.attachControl(this.canvas, true);
    }

    /* 创建场景*/
    createScene(engine: Engine): Scene {
        const canvas = engine.getRenderingCanvas();
        const scene = new Scene(engine);
        scene.clearColor.set(0.2, 0.2, 0.2, 1);
        this.createCamera(scene);
        const light = new HemisphericLight('light1', new Vector3(0, 1, 0), scene);
        light.intensity = 0.7;

        const pbr = new PBRMetallicRoughnessMaterial('pbr', scene);
        pbr.environmentTexture = CubeTexture.CreateFromPrefilteredData(environment, scene);

        const gridSize = 4;

        for (let i = 0; i < gridSize; i++) {
            for (let j = 0; j < 10; j++) {
                const sphereMat = pbr.clone('sphereMat');
                sphereMat.metallic = 0.1;
                sphereMat.roughness = (i / gridSize) / 3;
                sphereMat.baseColor = Color3.White().scale(1 - (j / 10));
                const sphere = Mesh.CreateSphere('sphere', 16, 0.2, scene);
                sphere.material = sphereMat;
                sphere.position.y = i * 0.3;
                sphere.position.x = 0.3;
                sphere.position.z = j * 0.4;

                const cubeMat = pbr.clone('cubeMat');
                cubeMat.metallic = 0.6;
                cubeMat.roughness = (i / gridSize) / 3;
                cubeMat.baseColor = Color3.White().scale(1 - (j / 10));
                const box = Mesh.CreateBox('box', 0.2, scene);
                box.material = cubeMat;
                box.position.y = i * 0.3;
                box.position.x = -0.3;
                box.position.z = j * 0.4;
            }
        }

        // Create default pipeline and enable dof with Medium blur level
        const pipeline = new DefaultRenderingPipeline('default', true, scene, [scene.activeCamera]);
        pipeline.depthOfFieldBlurLevel = DepthOfFieldEffectBlurLevel.Medium;
        pipeline.depthOfFieldEnabled = true;
        pipeline.depthOfField.focalLength = 180;
        pipeline.depthOfField.fStop = 3;
        pipeline.depthOfField.focusDistance = 2250;
        let moveFocusDistance = true;

        //add UI to adjust pipeline.depthOfField.fStop, kernelSize, focusDistance, focalLength
        const bgCamera = new ArcRotateCamera('BGCamera', Math.PI / 2 + Math.PI / 7, Math.PI / 2, 100, new Vector3(0, 20, 0), scene);
        bgCamera.layerMask = 0X40000000;
        const advancedTexture = AdvancedDynamicTexture.CreateFullscreenUI('UI');
        advancedTexture.layer.layerMask = 0X40000000;
        const UiPanel = new StackPanel();
        UiPanel.width = '220px';
        UiPanel.fontSize = '14px';
        UiPanel.horizontalAlignment = Control.HORIZONTAL_ALIGNMENT_RIGHT;
        UiPanel.verticalAlignment = Control.VERTICAL_ALIGNMENT_CENTER;
        advancedTexture.addControl(UiPanel);

        const params = [
            { name: 'fStop', min: 1.4, max: 32 },
            { name: 'focusDistance', min: 0, max: 5000 },
            { name: 'focalLength', min: 0, max: 500 }
        ];
        params.forEach((param) => {
            const header = new TextBlock();
            header.text = param.name + ':' + pipeline.depthOfField[param.name].toFixed(2);
            header.height = '40px';
            header.color = 'black';
            header.textHorizontalAlignment = Control.HORIZONTAL_ALIGNMENT_LEFT;
            header.paddingTop = 10;
            UiPanel.addControl(header);
            const slider = new Slider();
            slider.horizontalAlignment = Control.HORIZONTAL_ALIGNMENT_LEFT;
            slider.minimum = param.min;
            slider.maximum = param.max;
            slider.color = '#636e72';
            slider.value = pipeline.depthOfField[param.name];
            slider.height = '20px';
            slider.width = '205px';
            UiPanel.addControl(slider);
            slider.onValueChangedObservable.add((v) => {
                pipeline.depthOfField[param.name] = v;
                header.text = param.name + ':' + pipeline.depthOfField[param.name].toFixed(2);
                moveFocusDistance = false;
            });
        });

        scene.activeCameras = [scene.activeCamera, bgCamera];
        scene.onBeforeRenderObservable.add(() => {
            if (moveFocusDistance) {
                pipeline.depthOfField.focusDistance = 600 + (4000 * (Math.sin((new Date).getTime() / 1000) + 1) / 2);
            }
        });
        return scene;
    }

}

```
