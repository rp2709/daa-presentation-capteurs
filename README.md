# Capteurs Android
## Introduction
Les appareils androids portables possédent un assortiment de capteurs variés, dont la présence n'est pas garantie. 
Nous allons tout d'abord comprendre la méthodologie d'accès au capteurs avant d'explorer les trois catégories principales de capteurs et détailler leur fonctionnement et plus particulièrement les données qu'ils génèrent. Enfin, nous exploreront quelques examples de code Kotlin intégrant ces fonctionalités.

[Basé sur cette ressource](https://developer.android.com/develop/sensors-and-location/sensors/sensors_overview)
## Méthodologie
De part leur nature physique, les capteurs provoquent des événements, reçus et traités par un objet implémentant l'interface `SensorEventListener`. Cet objet doit être enregistré comme observateur d'événements pour un capteur au travers du `SensorManager` de la manière suivante
```kotlin
// Assume being in the context of an activity
// Initialization happen in the create method of activity
SensorManager manager = (SensorManager)getSystemService(SENSOR_SERVICE)
Sensor sensor = manager.getDefaultSensor(Sensor.TYPE_ACCELERATION_SENSOR);
SensorEventListener myEventListener;
int samplingPeriodUs = 10000; // => max frequency = 100Hz 

// onResume
manager.registerListener(myEventListener,sensor,10000)

// onPause
manager.unregisterListener(myEventListener)

```

**Il est crucial de retirer son objet de la liste des observeurs pour éviter de drainer la batterie. Quitter l'activité ne suffit pas.**  
Une période d'échantillonage plus petite va nous donner une meilleure approximation de la réalité mais requiert plus de temps sur le processeur et donc de batterie. Les applications sont par défault limitées à 200Hz (période de 5000Us), à moins de spécifier le contraire dans leur manifeste avec `Manifest.permission.HIGH_SAMPLING_RATE_SENSORS`.

Une fois enregistré comme observateur, le `SensorEventListener` réagit aux évènements suivants:
- onSensorChanged(SensorEvent event){...} : nouvel échantillon des données du capteur disponible au travers de `SensorEvent`
- onAccuracyChanged(Sensor sensor, int accuracy){...} : la précision du capteur a changé

On note que la méthode OnSensor**Changed**() est mal nommée, puisqu'elle sera appelée même si la valeur du capteur en elle même n'a pas changé. On considère le temps de la mesure comme ayant changé (duh).

La classe SensorManager va bien au-delà de l'accès brute aux capteurs individuels. Elle expose une grande collection de méthodes combinant les données de un ou plusieurs capteurs pour donner un résultat directement utilisable. On a entre autre:
- getRotationMatrix
- getOrientation
- getInclination
- getAngleChange
- getAltitude


## Les capteurs
### Capteurs de mouvement
Mesure des accélération linéaires et de rotation de l'appareil en trois dimensions.

#### Accélérometre
Mesure l'accélération sur les trois axes en **m/s²**. `TYPE_ACCELEROMETER` donne accès à des données prenant en compte une calibration. `TYPE_ACCELEROMETER_UNCALIBRATED` nous donne accès aux valeurs brutes sans correction.

Les valeurs relatives aux trois axes sont disponible au travers de l'objet sensorEvent.values[axe], avec axe égale à 0,1 ou 2 pour réspectivement x,y, ou z.  
Avec le type non-calibré, on a accès à deux jeux de valeurs, sans et avec compensation du bias, respectivement (0,1,2) et (3,4,5).

![axes](images/device-acceleration-coordinates.png)
[image_credit](https://google-developer-training.github.io/android-developer-advanced-course-concepts/unit-1-expand-the-user-experience/lesson-3-sensors/3-2-c-motion-and-position-sensors/3-2-c-motion-and-position-sensors.html)

Le `TYPE_GRAVITY` mesure uniquement l'accélération due à la gravité, à nouveau dans les trois axes. On accède aux valeurs de manière analogue.

Il est intéressant de noter que la plupart du temps, il n'y a qu'un seul capteur matériel, et que certains types de capteurs correspondent en fait à des capteurs logiciels ou conceptuels, comme un accéléromètre filtrant l'influence de la gravité ou un autre ne conservant que cette dernère.

[](https://developer.android.com/develop/sensors-and-location/sensors/sensors_motion)
[](https://developer.android.com/reference/android/hardware/SensorEvent#values)
[](https://developer.android.com/reference/android/hardware/Sensor#TYPE_ACCELEROMETER)

#### Gyroscope
Ce capteur fournit des données sur la rotation de l'appareil, plus précisément sur sa vitesse angulaire en radiants par seconde, ou s⁻¹ dans le SI. Pour obtenir la rotation, on intègre la vitesse angulaire par rapport au temps. Considèrons le code suivant, grandment simplifié:
```kotlin
float previous_sample_timestamp = 0;
float angle = 0; // angle in radiants

public void onSensorChangeEvent(SensorEvent event){
	if(previous_sample_timestamp == 0){
		previous_sample_timestamp = event.timestamp;
		return;
	}

	float angular_speed = event.values[0];
	float delta_time = event.timestamp - previous_sample_timestamp;
	angle += angular_speed * delta_time;
	previous_sample_timestamp = event.timestamp; 
}
```

Le code se complique pour trois dimensions, mais le principe d'intégration reste exactement le même.
[](https://developer.android.com/reference/android/hardware/SensorEvent#values)
[](https://developer.android.com/reference/android/hardware/Sensor#TYPE_GYROSCOPE)

### Capteurs d'environment
Mesure divers propriétés de l'environnement physique dans lequel l'appareil évolue, comme la température, la pression, l'humidité, ou encore la luminosité.

### Capteurs de position
Mesure de la position et de 'orientation géographique de l'appareil.

## Exemples

## Remarques
