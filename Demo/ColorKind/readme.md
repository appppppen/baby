### Use VertexBuffer.ColorKind

```typescript
const sphere = MeshBuilder.CreateIcoSphere('m', { radius: 2.0, updatable: true }, scene);
        
let colors = sphere.getVerticesData(VertexBuffer.ColorKind);
if (!colors) {
    colors = [];
    const positions = sphere.getVerticesData(VertexBuffer.PositionKind);
    for (let p = 0; p < positions.length / 3; p++) {
        colors.push(Math.random(), Math.random(), Math.random(), 1);
    }
}
sphere.setVerticesData(VertexBuffer.ColorKind, colors);
```
