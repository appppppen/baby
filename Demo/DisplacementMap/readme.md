### Use Mesh.applyDisplacementMap

```typescript
const displacementmapURL =
  "https://playground.babylonjs.com/textures/distortion.png";

const star = MeshBuilder.CreateSphere(
  "star",
  { diameter: 45, sideOrientation: Mesh.DOUBLESIDE, updatable: true },
  scene
);
star.applyDisplacementMap(displacementmapURL, 1, 250, null, null, null, true);
const texture = new StandardMaterial("texture", scene);
star.material = texture;
```
