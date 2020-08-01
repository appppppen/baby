### VertexBuffer.PositionKind

```typescript
const whirlpool = Mesh.CreateLines('whirlpool', points, scene, true);
whirlpool.color = new Color3(1, 1, 1);
const positionData = whirlpool.getVerticesData(VertexBuffer.PositionKind);
let colorData = whirlpool.getVerticesData(VertexBuffer.ColorKind);
const heightRange = 10;

let alpha = 0;
scene.registerBeforeRender(() => {
    colorData = [];
    for (let index = 0; index < 1000; index++) {
positionData[index * 3 + 1] = heightRange * Math.sin(alpha + index * 0.1);
// colorData[index * 3] = 0.5 * (Math.sin(alpha + index * 0.1) + 1);
colorData.push(Math.random(), Math.random(), Math.random(), 1);
    }

    whirlpool.updateVerticesData(VertexBuffer.PositionKind, positionData);
    whirlpool.updateVerticesData(VertexBuffer.ColorKind, colorData);
    alpha += 0.05 * scene.getAnimationRatio();
});
```
