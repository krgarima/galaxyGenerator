# threejs

All of Garima's threejs learning will be documented in this repo

# Three.js Journey

## Setup

Download [Node.js](https://nodejs.org/en/download/).
Run this followed commands:

```bash
# Install dependencies (only the first time)
npm install

# Run the local server at localhost:8080
npm run dev

# Build for production in the dist/ directory
npm run build
```

Galaxy Generator -

Base particles -

1. Create a generateGalaxy function. Each time we call that function, we will remove the previous galaxy (if there is one) and create a new one.

```
const generateGalaxy = () =>
{
    //...
}
generateGalaxy()
```

2. Create an object that will contain all the parameters that will be used later.

```
const parameters = {
  count: 100000,
  size: 0.05,
  radius: 6.5,
  branches: 6,
  spin: 1,
  randomness: 0.44,
  randomnessPower: 6,
  insideColor: "#f8691b",
  outsideColor: "#0b3293",
};
```

3. In our generateGalaxy function, we're going to create some particles just to make sure that everything is working. We can start with the geometry and add the particles count to the parameters:

```
const generateGalaxy = () =>
{
    /**
     * Geometry
     */
    const geometry = new THREE.BufferGeometry()

    const positions = new Float32Array(parameters.count * 3)

    for(let i = 0; i < parameters.count; i++)
    {
        const i3 = i * 3

        positions[i3    ] = (Math.random() - 0.5) * 3
        positions[i3 + 1] = (Math.random() - 0.5) * 3
        positions[i3 + 2] = (Math.random() - 0.5) * 3
    }

    geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
}
generateGalaxy()
```

4. Create the material by using the PointsMaterial class.

```
const generateGalaxy = () =>
{
    // ...

    /**
     * Material
     */
    const material = new THREE.PointsMaterial({
        size: parameters.size,
        sizeAttenuation: true,
        depthWrite: false,
        blending: THREE.AdditiveBlending
    })
}
```

5. Finally, we can create the points by using the Points class and add it to the scene:

```
const generateGalaxy = () =>
{
    // ...

    /**
     * Points
     */
    const points = new THREE.Points(geometry, material)
    scene.add(points)
}
```

<img width="566" alt="POINTS" src="https://github.com/Neuphony/threejs/assets/110389092/2160ba1c-0a65-483b-a064-6215315a3395">


6. Destroy old galaxy -
   We are creating galaxies one above the other.

   a. To make things right, we must first move the geometry, material and points variables outside the generateGalaxy.

   ```
   let geometry = null
   let material = null
   let points = null

   ```

   b. Then, before assigning those variables, we can test if they already exist. If so, we can call the dispose() method on the geometry and the material. Then remove the points from the scene with the remove() method:

   ```
   const generateGalaxy = () =>
   {
    // Destroy old galaxy
    if(points !== null)
    {
        geometry.dispose()
        material.dispose()
        scene.remove(points)
    }

    // ...
   }
   ```

7. Use lil-gui to change parameters in realtime (optional)

```
gui
  .add(parameters, "count")
  .min(100)
  .max(1000000)
  .step(100)
  .onFinishChange(galaxyGenerator);
gui
  .add(parameters, "size")
  .min(0.001)
  .max(0.1)
  .step(0.001)
  .onFinishChange(galaxyGenerator);
gui
  .add(parameters, "radius")
  .min(0.01)
  .max(20)
  .step(0.01)
  .onFinishChange(galaxyGenerator);
gui
  .add(parameters, "branches")
  .min(2)
  .max(20)
  .step(1)
  .onFinishChange(galaxyGenerator);
gui
  .add(parameters, "spin")
  .min(-5)
  .max(5)
  .step(0.001)
  .onFinishChange(galaxyGenerator);
gui
  .add(parameters, "randomness")
  .min(0)
  .max(2)
  .step(0.001)
  .onFinishChange(galaxyGenerator);
gui
  .add(parameters, "randomnessPower")
  .min(1)
  .max(10)
  .step(0.001)
  .onFinishChange(galaxyGenerator);
gui.addColor(parameters, "insideColor").onFinishChange(galaxyGenerator);
gui.addColor(parameters, "outsideColor").onFinishChange(galaxyGenerator);
```

To generate a new galaxy, you must listen to the change event. More precisely to the finishChange event to prevent generating galaxies while you are drag and dropping the range value.

SPIRAL GALAXIES -
We will now focus on spiral galaxies

RADIUS -

Each star will be positioned accordingly to that radius. If the radius is 5, the stars will be positioned at a distance from 0 to 5. For now, let's position all the particles on a straight line:

```
for(let i = 0; i < parameters.count; i++)
{
    const i3 = i * 3

    const radius = Math.random() * parameters.radius

    positions[i3    ] = radius
    positions[i3 + 1] = 0
    positions[i3 + 2] = 0
}
```

<img width="563" alt="Screenshot 2023-11-04 150434" src="https://github.com/Neuphony/threejs/assets/110389092/6299089b-0935-4edd-8045-f664f85fa5c8">


BRANCHES -

We can use Math.cos(...) and Math.sin(...) to position the particles on those branches. We first calculate an angle with the modulo (%), divide the result by the branches count parameter to get an angle between 0 and 1, and multiply this value by Math.PI \* 2 to get an angle between 0 and a full circle. We then use that angle with Math.cos(...) and Math.sin(...) for the x and the z axis and we finally multiply by the radius:

```
for(let i = 0; i < parameters.count; i++)
{
    const i3 = i * 3

    const radius = Math.random() * parameters.radius
    const branchAngle = (i % parameters.branches) / parameters.branches * Math.PI * 2

    positions[i3    ] = Math.cos(branchAngle) * radius
    positions[i3 + 1] = 0
    positions[i3 + 2] = Math.sin(branchAngle) * radius
}
```

<img width="564" alt="Screenshot 2023-11-04 151000" src="https://github.com/Neuphony/threejs/assets/110389092/1f622a92-6e29-4108-b0c3-c7ef4bb7eaff">


SPIN -

Let's add the spin effect.
We can multiply the spinAngle by that spin parameter. To put it differently, the further the particle is from the center, the more spin it'll endure:

```
for(let i = 0; i < parameters.count; i++)
{
    const i3 = i * 3

    const radius = Math.random() * parameters.radius
    const spinAngle = radius * parameters.spin
    const branchAngle = (i % parameters.branches) / parameters.branches * Math.PI * 2

    positions[i3    ] = Math.cos(branchAngle + spinAngle) * radius
    positions[i3 + 1] = 0
    positions[i3 + 2] = Math.sin(branchAngle + spinAngle) * radius
}
```

<img width="960" alt="spin 3" src="https://github.com/Neuphony/threejs/assets/110389092/56a3c06e-78f9-4d08-9f26-f01f7119de88">

RANDOMNESS -

Those particles are perfectly aligned. We need randomness. But what we truly need is spread stars on the outside and more condensed star on the inside.

create a random value for each axis with Math.random(), multiply it by the radius and then add those values to the positions:

```
for(let i = 0; i < parameters.count; i++)
{
    const i3 = i * 3

    const radius = Math.random() * parameters.radius

    const spinAngle = radius * parameters.spin
    const branchAngle = (i % parameters.branches) / parameters.branches * Math.PI * 2

    const randomX = (Math.random() - 0.5) * parameters.randomness * radius
    const randomY = (Math.random() - 0.5) * parameters.randomness * radius
    const randomZ = (Math.random() - 0.5) * parameters.randomness * radius

    positions[i3    ] = Math.cos(branchAngle + spinAngle) * radius + randomX
    positions[i3 + 1] = randomY
    positions[i3 + 2] = Math.sin(branchAngle + spinAngle) * radius + randomZ
}
```
<img width="960" alt="random galaxy" src="https://github.com/Neuphony/threejs/assets/110389092/b95bedfe-f769-4f03-b2f3-724f9fdc68b5">

It's working but it's not very convincing, right? And we can still see the pattern. To fix that, we can use Math.pow() to crush the value. The more power you apply, the closest to 0 it will get. The problem is that you can't use a negative value with Math.pow(). What we will do is calculate the power then multiply it by -1 randomly.
Apply the power with Math.pow() and multiply it by -1 randomly:

```
const randomX = Math.pow(Math.random(), parameters.randomnessPower) * (Math.random() < 0.5 ? 1 : - 1) * parameters.randomness * radius
const randomY = Math.pow(Math.random(), parameters.randomnessPower) * (Math.random() < 0.5 ? 1 : - 1) * parameters.randomness * radius
const randomZ = Math.pow(Math.random(), parameters.randomnessPower) * (Math.random() < 0.5 ? 1 : - 1) * parameters.randomness * radius
```
<img width="960" alt="power randomness spiral" src="https://github.com/Neuphony/threejs/assets/110389092/a6bb8f05-e36e-4c8e-a243-ffc4ef96afc9">



COLORS -

We're going to provide a color for each vertex. We must active the vertexColors on the material:

```
material = new THREE.PointsMaterial({
    size: parameters.size,
    sizeAttenuation: true,
    depthWrite: false,
    blending: THREE.AdditiveBlending,
    vertexColors: true
})
```

Then add a color attribute on our geometry just like we added the position attribute. For now, we're not using the insideColor and outsideColor parameters:

```
geometry = new THREE.BufferGeometry()

const positions = new Float32Array(parameters.count * 3)
const colors = new Float32Array(parameters.count * 3)

for(let i = 0; i < parameters.count; i++)
{
    // ...

    colors[i3    ] = 1
    colors[i3 + 1] = 0
    colors[i3 + 2] = 0
}

geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3))
```

<img width="960" alt="spiral color" src="https://github.com/Neuphony/threejs/assets/110389092/fc3f2ac2-32da-438c-a3c6-3abb7b9db966">


Mix Color -

First need to create a Color instance for each one.

```
const generateGalaxy = () =>
{
    // ...

    const colorInside = new THREE.Color(parameters.insideColor)
    const colorOutside = new THREE.Color(parameters.outsideColor)

    // ...
}
```

Inside the loop function, we want to mix these colors into a third color. That mix depends on the distance from the center of the galaxy. If the particle is at the center of the galaxy, it'll have the insideColor and the further it gets from the center, the more it will get mixed with the outsideColor.
Instead of creating a third Color, we are going to clone the colorInside and then use the lerp(...) method to interpolate the color from that base color to another one. The first parameter of lerp(...) is the other color, and the second parameter is a value between 0 and 1. If it's 0, the color will keep its base value, and if it's 1 the result color will be the one provided. We can use the radius divided by the radius parameter:

```
const mixedColor = colorInside.clone()
mixedColor.lerp(colorOutside, radius / parameters.radius)
```

We can then use the r, g and b properties in our colors array:

```
colors[i3    ] = mixedColor.r
colors[i3 + 1] = mixedColor.g
colors[i3 + 2] = mixedColor.b
```

https://github.com/Neuphony/threejs/assets/110389092/7af352c7-ac24-4809-bc73-78d7bf3686a1


