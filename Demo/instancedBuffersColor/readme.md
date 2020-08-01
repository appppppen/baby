### instancedBuffers.color

```typescript
const box = BoxBuilder.CreateBox('root', { size: 2 });
box.alwaysSelectAsActiveMesh = true;
const instanceCount = 10000;
const mat = new StandardMaterial('material', scene);
mat.disableLighting = true;
mat.emissiveColor = Color3.White();
box.material = mat;

box.registerInstancedBuffer('color', 4);
box.instancedBuffers.color = new Color4(Math.random(), Math.random(), Math.random(), 1);

const baseColors = [];
const alphas = [];

for (let index = 0; index < instanceCount - 1; index++) {
    const instance = box.createInstance('box' + index);
    instance.position.x = 20 - Math.random() * 40;
    instance.position.y = 20 - Math.random() * 40;
    instance.position.z = 20 - Math.random() * 40;
    instance.alwaysSelectAsActiveMesh = true;
    alphas.push(Math.random());
    baseColors.push(new Color4(Math.random(), Math.random(), Math.random(), 1));
    instance.instancedBuffers.color = baseColors[baseColors.length - 1].clone();
}

scene.registerBeforeRender(() => {
    for (let instanceIndex = 0; instanceIndex < box.instances.length; instanceIndex++) {
let alpha = alphas[instanceIndex];
const cos = Math.abs(Math.cos(alpha));
alpha += 0.01;
alphas[instanceIndex] = alpha;
const instance = box.instances[instanceIndex];
baseColors[instanceIndex].scaleToRef(cos, instance.instancedBuffers.color);
    }
});
```
