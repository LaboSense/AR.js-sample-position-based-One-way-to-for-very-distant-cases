# AR.js-sample-position-based-One-way-to-for-very-distant-cases

AR.js is a great project.  I am deeply grateful to all involved.
However, when building location-based AR, objects that are more than about 1km away will not be displayed even if the scale is increased.

The official suggestion is to set the videoTexture parameter to true.
[Ar.js Document](https://ar-js-org.github.io/AR.js-Docs/location-based/ "Viewing every distant object")

However, on my iPhone (iOS14), video autoplay is not allowed by permissions and the screen freezes.
Objects are displayed, but the screen is frozen, so it is not AR.

I was puzzled.

So I decided to place pseudo-distant objects within 1km of each other.
Using geodesy([chrisveness/geodesy](https://github.com/chrisveness/geodesy "chrisveness/geodesy")), we find the direction of the current location and the set position, as well as the latitude and longitude of the 800m point in that direction.

geodesy is very nice too, thanks.

It's only pseudo, but it can display any position, no matter how far away it is.

When multiple objects are placed, the display is controlled by adding 1/1000th of the actual distance to 800m. The scale is specified according to the distance, and pseudo-distant objects are made smaller.

As I have said many times, this is not a fundamental solution, but it has served our purpose.

Thank you.

###### How to use
Add the latitude and longitude you want to specify in the next part.
```calc.js
function staticLoadPlaces() {
    return [
        {
            name: 'Time Desk',
            location: {
                lat: 43.062533,
                lng: 141.353638,
            }
        },
      // Follow this part add position.
    ];
}
```

If you want to use gftf, you can change it as follows.

calc.js add model name to staticLoadPlaces.
```calc.js
function staticLoadPlaces() {
    return [
        {
            name: 'Time Desk',
            modelName: 'assets.gltf',
            location: {
                lat: 43.062533,
                lng: 141.353638,
            }
        },
    ];
}
```
Change the display section as follows.
assets directory contains gltf files.
```
function renderPlaces(places, pos) {
    let scene = document.querySelector('a-scene');
    var crd = pos.coords;
    let cal = new CalcVR();

    places.forEach((place) => {
        let latitude = place.location.lat;
        let longitude = place.location.lng;
        let name = place.name;
        let modelName = place.modelName;
        cal.calcDist([crd.latitude, crd.longitude], [latitude, longitude]);
        cal.calcNewPosition(cal.currentPosition, cal.bearing, cal.newDistance);
        cal.calcSizeDist(cal.distance);
        let model = document.createElement('a-entity');
        model.setAttribute('look-at', '[gps-camera]');
        model.setAttribute('gps-entity-place', `latitude: ${cal.newPosition[0]}; longitude: ${cal.newPosition[1]};`);
        model.setAttribute('gltf-model', `./assets/${modelName}`);
        model.setAttribute('animation-mixer', '');
        model.setAttribute('scale', `${cal.objectSize}`);

        model.addEventListener('loaded', () => {
            window.dispatchEvent(new CustomEvent('gps-entity-place-loaded'))
        });

        scene.appendChild(model);
    });
}
```
I hope this helps.
