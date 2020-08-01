### Use Path3D

```typescript
import { BaseScene } from '../../../../babylon/template/Base/BaseScene';
import {
    ArcRotateCamera, Vector3, HemisphericLight, Engine, Scene, MeshBuilder, Color3, Path3D
} from '@babylonjs/core/Legacy/legacy';

export class AssembleScene extends BaseScene {
    newVector: Vector3;

    createScene(engine: Engine): Scene {
        const canvas = engine.getRenderingCanvas();
        const scene = new Scene(engine);
        scene.clearColor.set(0.2, 0.2, 0.2, 1);

        const light = new HemisphericLight('light1', new Vector3(0, 1, 0), scene);
        light.intensity = 0.3;
        const camera = new ArcRotateCamera('Camera', 0, 0, 50, Vector3.Zero(), scene);
        camera.lowerRadiusLimit = 10;
        camera.upperRadiusLimit = 100;
        camera.setPosition(new Vector3(0, 350, 350));
        camera.attachControl(this.canvas, false);
        scene.activeCameras.push(camera);

        const points = [];
        for (let i = 0; i < 50; i++) {
            points.push(new Vector3(i - 25, 5 * Math.sin(i / 2), i / 5 * Math.cos(i / 2)));
        }

        const path3d = new Path3D(points);
        let tangents = path3d.getTangents();
        let normals = path3d.getNormals();
        let binormals = path3d.getBinormals();
        const curve = path3d.getCurve();

        let li = MeshBuilder.CreateLines('li', { points: curve, updatable: true }, scene);
        const tg = [];
        const no = [];
        const bi = [];
        for (let p = 0; p < curve.length; p++) {
            tg[p] = MeshBuilder.CreateLines('tg', { points: [curve[p], curve[p].add(tangents[p])], updatable: true }, scene);
            tg[p].color = Color3.Red();
            no[p] = MeshBuilder.CreateLines('no', { points: [curve[p], curve[p].add(normals[p])], updatable: true }, scene);
            no[p].color = Color3.Blue();
            bi[p] = MeshBuilder.CreateLines('bi', { points: [curve[p], curve[p].add(binormals[p])], updatable: true }, scene);
            bi[p].color = Color3.Green();
        }

        let theta = 0;

        scene.registerAfterRender(() => {
            theta += 0.05;
            this.newVector = new Vector3(Math.cos(theta), 0, Math.sin(theta));
            path3d.update(curve, this.newVector);
            tangents = path3d.getTangents();
            normals = path3d.getNormals();
            binormals = path3d.getBinormals();
            li = MeshBuilder.CreateLines('li', { points: curve, instance: li, updatable: true });
            for (let p = 0; p < curve.length; p++) {
                no[p] = MeshBuilder.CreateLines('no',
                    {
                        points: [curve[p], curve[p].add(normals[p])],
                        instance: no[p],
                        updatable: true
                    }, scene);

                bi[p] = MeshBuilder.CreateLines('bi',
                    {
                        points: [curve[p], curve[p].add(binormals[p])],
                        instance: bi[p],
                        updatable: true
                    }, scene);
            }
        });
        return scene;
    }

}
```
