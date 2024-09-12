# 16 SHADOWS 
* dark shadows in the back of the objects are called **core shadows** 
* currently missing **drop shadows**
* we want to see the sphere silhouette shadow
* shadows have always been a challenge for **real-time 3D rendering** and devs need to find tricks to display realistics shadows at a reasonable frame rate
* Three.js has a built-in solution
* not perfect but convenient 
## HOW IT WORKS
* when you do one render, Three.js will do a render for each light supporting shadows
* those renders will simulate what the light sees as if it was a camera
* during these lights renders, a `MeshDepthMaterial` replaces all meshes materials 
* light renders are stores as textures and we call those **shadow maps** 
* they are then used on every material supposed to receive shadows and projected on the geometry 
## SETUP 
* a sphere
* a plane
* a directional light
* an ambient light (to simulate light bounce)
## HOW TO ACTIVATE SHADOWS 
* activate the shadow maps on the `renderer`:
    ```
    renderer.shadowMap.enabled = true
    ```
* go through each object and decide if it can cast a shadow with `castShadow` and if it can receive shadow with `receiveShadow`:
    ```
    // sphere
    sphere.castShadow = true 
    // plane
    plane.receiveShadow = true
    ```
* only the following types of lights support shadows:
    - `PointLight`
    - `DirectionalLight`
    - `SpotLight`
* activate the shadows on the light with the `castShadow`:
    ```
    // Directional Light
    directionalLight.castShadow = true 
    ```
## SHADOW MAP OPTIMISATIONS 
### RENDER SIZE (SHADOW MAP SIZE)
* can access the shadow map in the `shadow` property of each light:
    ```
    console.log(directionalLight.shadow)
    ```
* by default the shadow map size is `512x512`
* we can improve this but must be a power of 2 for `mipmapping`:
    ```
    directionalLight.shadow.mapSize.width = 1024
    directionalLight.shadow.mapSize.height = 1024
    ```
### NEAR AND FAR
* Three.js is using camera to do the shadow maps renders
* these cameras have the sam eproperties like `near` and `far`
* to help debug can use `CameraHelper` wwith the camera used for the shadow map located in the `directionalLight.shadow.camera`:
    ```
    const directionalLightCameraHelper = new THREE.CameraHelper(directionalLight.shadow.camera)
    scene.add(directionalLightCameraHelper)
    ```
* can now find decent `near` and `far`:
    ```
    directionalLight.shadow.camera.near = 1
    directionalLight.shadow.camera.far = 6
    ```
### AMPLITUDE 
* with the camera helper we can see that the amplitude is too large
* b/c using a `DirectionalLight` Three.js is using an `OrthographicCamera`
* can control how far on each side the camera can see with `top`, `right`, `bottom`, and `left`:
    ```
    directionalLight.shadow.camera.top = 2
    directionalLight.shadow.camera.right = 2
    directionalLight.shadow.camera.bottom = -2
    directionalLight.shadow.camera.left = -2
    ```
* the smaller the values, the more precise the shadow will be 
* if it's too small, the shadows will be cropped 
* hide the camera helper:
    ```
    directionalLightCameraHelper.visible = false
    ```
### BLUR
* can control the shadow blur with the `radius` property:
    ```
    directionalLight.shadow.radius = 10
    ```
* this technique doesn't use the proximity of the camera with the object, it's a general and cheap blur
### SHADOW MAP ALGORITHM 
* different types of algorithms can be applied to shadow maps:
    - `THREE.BasicShadowMap` - great performance but bad quality 
    - `THREE.PCFShadowMap` - less performance but smoother edges (default)
    - `THREE.PCFSoftShadowMap` - worse performance but even softer edges (usually go for this one)
    - `THREE.VSMShadowMap` - worse performance, more constraints, can have unexpected results 
* update the `renderer.shadowMap.type`:
    ```
    renderer.shadowMap.type = THREE.PCFSoftShadowMap 
    ```
* **radius** doesn't work with `THREE.PCFSoftShadowMap`
## SPOTLIGHT
* add a `SpotLight`:
    ```
    const spotLight = new THREE.SpotLight(0xffffff, 0.4, 10, Math.PI * 0.3)

    spotLight.castShadow = true

    spotLight.position.set(0, 2, 2)
    scene.add(spotLight)
    scene.add(spotLight.target)
    ```
* add a camera helper:
    ```
    const spotLightCameraHelper = new THREE.CameraHelper(spotLight.shadow.camera)
    scene.add(spotLightCameraHelper)
    ```
* reduce the other lights intensity:
    ```
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.4)
    // ...
    const directionalLight = new THREE.DirectionalLight(0xffffff, 0.4)
    ```
* mixing shadows doesn't look good and theres not much you can do it about it
* can improve the shadow quality using the same techniques applied to the directional light 
### RENDER SIZE (SHADOW MAP SIZE)
```
spotLight.shadow.mapSize.width = 1024
spotLight.shadow.mapSize.height = 1024
```
### AMPLITUDE 
* b/c using `SpotLight`, Three.js is using a `PerspectiveCamera` 
* must change `fov` property to adapt the amplitude 
    ```
    spotLight.shadow.camera.fov = 30
    ```
### NEAR AND FAR 
```
spotLight.shadow.camera.near = 1 
spotLight.shadow.camera.far = 6
```
* hide the camera helper:
    ```
    spotLightCameraHelper.visible = false 
    ```
## POINTLIGHT 
* add a `PointLight`
    ```
    const pointLight = new THREE.PointLight(0xffffff, 0.3)

    pointLight.castShadow = true

    pointLight.position.set(-1, 1, 0)
    scene.add(pointLight)
    ```
* add camera helper:
    ```
    const pointLightCameraHelper = new THREE.CameraHelper(pointLight.shadow.camera)
    scene.add(pointLightCameraHelper)
    ```
* reduce other lights intensity:
    ```
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.3)

    // ... 
    const directionalLight = new THREE.DirectionalLight(0xffffff, 0.3)

    // ...
    const spotLight = new THREE.SpotLight(0xffffff, 0.3, 10, Math.PI * 0.3)
    ```
* camera helper seems to be a `PerspectiveCamera` facing downward
* Three.js uses a `PerspectiveCamera` but in all 6 directions and finished downward
### RENDER SIZE (SHADOW MAP SIZE)
```
pointLight.shadow.mapSize.width = 1024
pointLight.shadow.mapSize.height = 1024
```
### NEAR AND FAR 
```
pointLight.shadow.camera.near = 0.1
pointLight.shadow.camera.far = 5
```
* hide camera helper:
    ```
    pointLightCameraHelper.visible = false 
    ```
## BAKING SHADOWS 
* a good alternative to Three.js Journey shadows is **baked shadows** 
* intergrate shadows in textures that are applied to materials 
* deactivate all shadows on the lights:
    ```
    directionalLight.castShadow = false 

    // ...
    spotLight.castShadow = false

    // ...
    pointLight.castShadow = false
    ```
* load the shadow:
    ```
    const textureLoader = new THREE.TextureLoader()
    const bakedShadow = textureLoader.load('/textures/bakedShadow.jpg')
    ```
* can `console.log` to make sure no mistakes 
    ```
    console.log(bakedShadow)
    ```
* instead of `MeshStandardMaterial` use `MeshBasicMaterial` on the plane material with the `bakedShadow`:
    ```
    const plane = new THREE.Mesh(
        new THREE.PlaneGeometry(5, 5),
        new THREE.MeshBasicMaterial({
            map: bakedShadow
        })
    )
    ```
* it's not dynamic (so can't move sphere b/c shadow baked on plane)
## BAKING SHADOWS ALTERNATIVE
* can also use a more simpler baked shadow and move it so it stays under the sphere
* put the `MeshStandardMaterial` back on the plane:
    ```
    const plane = new THREE.Mesh(
        new THREE.PlaneGeometry(5, 5),
        material
    )
    ```
* load the basic shadow texture:
    ```
    const simpleShadow = textureLoader.load('/textures/simpleShadow.jpg')
    ```
* console.log to check 
    ```
    console.log(simpleShadow)
    ```
* create a plane slightly above (if you put 2 planes on the same level, can get a glitch called z fighting b/c the GPU doesn't know which plane should be seen) the floor with an `alphaMap` using the `simpleShadow`:
    ```
    const sphereShadow = new THREE.Mesh(
        new THREE.PlaneGeometry(1.5, 1.5),
        new THREE.MeshBasicMaterial({
            color: 0x000000,
            transparent: true,
            alphaMap: simpleShadow
        })
    )
    sphereShadow.rotation.x = - Math.PI * 0.5
    sphereShadow.position.y = plane.position.y + 0.01
    scene.add(sphereShadow)
    ```
* animate the sphere in the `tick` function:
    ```
    const clock = new THREE.Clock()

    const tick = () =>
    {
        const elapsedTime = clock.getElapsedTime()

        // Update the sphere 
        sphere.position.x = Math.cos(elapsedTime) * 1.5
        sphere.position.z = Math.sin(elapsedTime) * 1.5
        sphere.position.y = Math.abs(Math.sin(elapsedTime * 3))

        // ...
    }
    ```
* animate the shadow in the `tick` function:
```
const tick = () =>
{
    // ...

    // Update the shadow
    sphereShadow.position.x = sphere.position.x
    sphereShadow.position.z = sphere.position.z
    sphereShadow.material.opacity = (1 - sphere.position.y) * 0.3

    // ...
}
```
## WHICH TECHNIQUE TO USE?
* finding the right solution to handle shadows is up to you
* depends on the project, the performances and the techniques you know
* can combine techniques 
