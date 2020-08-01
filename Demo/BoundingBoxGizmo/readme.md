### Use BoundingBoxGizmo

```typescript
SceneLoader.LoadAssetContainer(Tools.GetFolderPath(alien), Tools.GetFilename(alien), scene, (container) => {
            const manager = new GUI3DManager(scene);
            container.addAllToScene();
            container.meshes[0].scaling.scaleInPlace(1.5);
            const gltfMesh = container.meshes[0];
            const boundingBox = BoundingBoxGizmo.MakeNotPickableAndWrapInBoundingBox(gltfMesh as Mesh);
            boundingBox.rotateAround(new Vector3(0, 0, 0), new Vector3(0, 1, 0), Math.PI);
            boundingBox.position.y = 0;

            const utilLayer = new UtilityLayerRenderer(scene);
            utilLayer.utilityLayerScene.autoClearDepthAndStencil = false;
            const gizmo = new BoundingBoxGizmo(Color3.FromHexString('#0984e3'), utilLayer);
            gizmo.attachedMesh = boundingBox;

            const sixDofDragBehavior = new SixDofDragBehavior();
            boundingBox.addBehavior(sixDofDragBehavior);
            const multiPointerScaleBehavior = new MultiPointerScaleBehavior();
            boundingBox.addBehavior(multiPointerScaleBehavior);

            const appBar = new TransformNode('');
            appBar.scaling.scaleInPlace(0.2);
            const panel = new PlanePanel();
            panel.margin = 0;
            panel.rows = 1;
            manager.addControl(panel);
            panel.linkToTransformNode(appBar);
            
            const button = new HolographicButton('orientation');
            panel.addControl(button);
            button.text = 'Button #' + panel.children.length;
            button.onPointerClickObservable.add(() => {
                if (gizmo.attachedMesh) {
                    gizmo.attachedMesh = null;
                    boundingBox.removeBehavior(sixDofDragBehavior);
                    boundingBox.removeBehavior(multiPointerScaleBehavior);
                } else {
                    gizmo.attachedMesh = boundingBox;
                    boundingBox.addBehavior(sixDofDragBehavior);
                    boundingBox.addBehavior(multiPointerScaleBehavior);
                }
            });
            const behavior = new AttachToBoxBehavior(appBar);
            boundingBox.addBehavior(behavior);
        });
```
