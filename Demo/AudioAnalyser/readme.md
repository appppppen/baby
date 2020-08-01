### Use Analyser

```typescript
const music = new Sound("Music", musicSrc, scene, null, {
  streaming: true,
  autoplay: true,
});

// Start the analyser
const myAnalyser = new Analyser(scene);
(Engine.audioEngine as AudioEngine).connectToAnalyser(myAnalyser);
// myAnalyser.connectAudioNodes()
myAnalyser.FFT_SIZE = 512;
myAnalyser.SMOOTHING = 0.9;

scene.registerBeforeRender(() => {
  this.uint8Array = myAnalyser.getByteFrequencyData();
  for (let i = 0; i < this.bar.length; i++) {
    this.bar[i].scaling.z = this.uint8Array[i] * 4;
  }
});
```
