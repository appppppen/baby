```typescript
import Vue from 'vue';
import { LinesMesh, Vector3, Color3, Engine, Scene } from '@babylonjs/core/Legacy/legacy';

import { ViewModel } from '../ViewModel';
import { Coordinate2DSystem } from '../../../../babylon/Math/utils/Coordinate2DSystem';
import { LinesBuild } from '../../../../babylon/util/LinesBuild';
import { Base2DScene } from '../../../../babylon/template/Base2DScene';
import { Vector3Utils } from '../../../../babylon/util/Vector3Utils';

export class AssembleScene extends Base2DScene {
    viewModel: ViewModel;
    edgesWidth = 6;
    HexWhite = '#ffffff';
    sys: Coordinate2DSystem;
    colorWhite: Color3;
    heart: LinesMesh;

    constructor(vm: Vue) {
        super();
        this.viewModel = vm as ViewModel;
        this.init();
    }

    resize() {
        this.changeCameraSize();
        super.resize();
    }

    /**
     * 初始化数值
     * @param engine 
     */
    initValue(scene: Scene) {
        this.colorWhite = Color3.FromHexString(this.HexWhite);
        this.sys = new Coordinate2DSystem('sys', scene)
            .create2DSystem(3, 1, Color3.FromHexString('#6f6f6f'), this.edgesWidth, 0, 0.2, 0.1);
    }

    /**
     * 创建场景
     * @param engine 
     */
    createScene(engine: Engine): Scene {
        const canvas = engine.getRenderingCanvas();
        const scene = new Scene(engine);
        scene.clearColor.set(0.2, 0.2, 0.2, 1);
        this.createTargetCamera4Math(scene, 3.5);
        this.camera.position.x = 0;
        this.initValue(scene);
        this.heart = LinesBuild.CreateUpdateLines(this.getHeartVertices(1), Color3.Red(), 6, this.heart, scene);
        this.reset();
        return scene;
    }


    getHeartPoint(x: number, t: number) {
        const a = Math.pow(x, 0.66);
        const b = 3.3 - Math.pow(x, 2);
        const y = a + 0.9 * Math.pow(b, 0.5) * Math.sin(t * x * Math.PI);
        return new Vector3(x, y, 0);
    }

    getHeartPointF(x: number, t: number) {
        const a = Math.pow(x, 0.66);
        const b = 3.3 - Math.pow(x, 2);
        const y = a + 0.9 * Math.pow(b, 0.5) * Math.sin(t * x * Math.PI);
        return new Vector3(-x, y, 0);
    }

    getHeartVertices(value: number) {
        const vertices = [];
        for (let i = 5; i > 0; i -= 0.001) {
            vertices.push(this.getHeartPointF(i, value));
        }
        for (let i = 0; i <= 5; i += 0.001) {
            vertices.push(this.getHeartPoint(i, value));
        }
        return vertices;
    }

    formatter(e: number) {
        this.updateMeshVertData(this.heart, Vector3Utils.ToArray(this.getHeartVertices(e)));
    }

    /**
     * 重置按钮按下
     */
    reset(): void {
    }
}

```