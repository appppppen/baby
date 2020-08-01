### Use scene.clipPlane

```typescript
const sphere = Mesh.CreateSphere("sphere1", 16, 2, scene);
sphere.position.y = 1;
sphere.onBeforeRenderObservable.add(() => {
  scene.clipPlane = new Plane(1, 0, 0, 0);
});

sphere.onAfterRenderObservable.add(() => {
  scene.clipPlane = null;
});
```
