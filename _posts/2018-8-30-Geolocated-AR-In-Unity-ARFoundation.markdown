---
layout: post
title: Geolocated AR in Unity ARFoundation
categories: unity C# AR ARCore
---

ARFoundation is an experimental package from unity that provides a common layer on top of Apple ARCore and Apple ARKit https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@1.0/manual/index.html


## Unity world origin and orientation
This point is fixed in the AR environment. To preserve accuracy and simplicity in dealing with the AR libraries, this will be aligned with the users starting GPS location.

## User location and heading
The user is moving around the physical world. Their position can be tracked with the phone GPS and compass. As inertial tracking and camera tracking are already used by the AR libraries, we will interact with them through the unity AR camera, not directly. The users GPS location will be stored on startup and used to offset all other GPS coordinates to align with the Unity World Origin

## AR Camera location and heading
This is tracked by the AR Libraries, and not directly controlled by our app. This position will be an offset from origin position determined when the application was started. The heading is not important at this time.

## Object location and heading
Location of objects in the real world, and the direction they're facing. These coordinates will be mapped into the Unity world space when they are displayed.

# Transforming GPS locations into Unity

User location can be provided by Unity's built in Location Service https://docs.unity3d.com/ScriptReference/LocationService.html, which interfaces with the device GPS.

Note: If debugging using Unity Remote https://docs.unity3d.com/Manual/UnityRemote5.html, the GPS service can take a significant time to startup, so the application will need to wait and retry until until GPS is established.

We wrap the GPS access in a class to handle the startup, polling and provides a fake location mode for local testing.

{% highlight csharp %}
public class GPSManager_NoCompass : MonoBehaviour
{
    public static GPSManager_NoCompass Instance { set; get; }
    public float latitude;
    public float longitude;

    public bool UseFakeLocation;

    [HideInInspector]
    public bool isRunning = true;

    [HideInInspector]
    public LocationServiceStatus ServiceStatus = LocationServiceStatus.Stopped;

    private void Start()
    {
        Instance = this;
        DontDestroyOnLoad(gameObject);
        StartCoroutine(StartLocationService());
    }

    private IEnumerator StartLocationService()
    {
        ServiceStatus = LocationServiceStatus.Initializing;
        // Allow a fake location to be returned when testing on a device that doesn't have GPS
        if (UseFakeLocation)
        {
            Debug.Log(string.Format("Using fake GPS location lat:{0} lon:{1}", latitude, longitude));
            ServiceStatus = LocationServiceStatus.Running;
            yield break;
        }

        if (!Input.location.isEnabledByUser)
        {
            Debug.Log("user has not enabled gps");
            yield break;
        }

        // Wait for the GPS to start up so there's time to connect
        Input.location.Start();

        yield return new WaitForSeconds(5);

        int maxWait = 20;
        while (Input.location.status == LocationServiceStatus.Initializing && maxWait > 0)
        {
            yield return new WaitForSeconds(1);
            maxWait--;
        }

        if (maxWait <= 0)
        {
            Debug.Log("Timed Out");
            yield break;
        }

        // If gps hasn't started by now, just give up
        ServiceStatus = Input.location.status;
        if (Input.location.status == LocationServiceStatus.Failed)
        {
            Debug.Log("Unable to determine device location");
            yield break;
        }

        //Loop forever to get GPS updates
        while (isRunning)
        {
            yield return new WaitForSeconds(2);
            UpdateGPS();
        }
    }

    private void UpdateGPS()
    {
        if (Input.location.status == LocationServiceStatus.Running)
        {
            latitude = Input.location.lastData.latitude;
            longitude = Input.location.lastData.longitude;
            ServiceStatus = Input.location.status;

            Debug.Log(string.Format("Lat: {0} Long: {1}", latitude, longitude));
        }
        else
        {
            Debug.Log("GPS is " + Input.location.status);
        }
    }
}
{% endhighlight %}

Now that we have the position,we need to convert from real world coordinates into unity AR world coordindates. Conversion between cartesian coordinate systems involes scaling and rotation transforms. As the earth is roughly spherical, the arc length of a degree of longitude changes with the latitude, approaching zero at the pole. This can be calculated by `111319.9 * Math.Cos(latitude * (Math.PI/180))`. As the earth is not a perfect sphere, the distance between degrees latitude also varies slightly, but an insignificant amount compared to logitude, so I will use the approximation of 111132m per degree.

![Coordinate Grids](/assets/CoordGrid1.png)

{% highlight csharp %}
private float GetLongitudeDegreeDistance(float latitude)
{
    return degreesLongitudeInMetersAtEquator * Mathf.Cos(latitude * (Mathf.PI / 180));
}

void SpawnObject () {
    // Real world position of object. Need to update with something near your own location.
    float latitude = -27.469093;
    float longitude = 153.023394;

    // Conversion factors
    float degreesLatitudeInMeters = 111132;
    float degreesLongitudeInMetersAtEquator = 111319.9f;

    // Real GPS Position - This will be the world origin.
    var gpsLat = GPSManager.Instance.latitude;
    var gpsLon = GPSManager.Instance.longitude;
    // GPS position converted into unity coordinates
    var latOffset = (latitude - gpsLat) * degreesLatitudeInMeters;
    var lonOffset = (longitude - gpsLon) * GetLongitudeDegreeDistance(latitude);

    // Create object at coordinates
    var obj = GameObject.CreatePrimitive(PrimitiveType.Cylinder);
    obj.transform.position = new Vector3(latOffset, 0, lonOffset);
    obj.transform.localScale = new Vector3(4, 4, 4);
}
{% endhighlight %}

To then align the GPS real world coordinate space with the Unity AR space, we can sample the devices compass to determine there direction it is facing when when AR is initialised. This rotation can then be applied when doing the coordinate transform to align both spaces.

The heading can be read in Unity by using `Input.compass.trueHeading`. This rotation can then be used to transform a position on the GPS coordinate system to be aligned with the unity coordinates.

{% highlight csharp %}
var heading = MathHelper.DegreesToRadians(GPSManager.heading);
// Need to rotate the the the offset to align to the world coords

var compassRotation = Quaternion.FromEulerAngles(0, heading, 0);
var rotatedOffset = compassRotation * offset;
{% endhighlight %}

The code then can be tidied up to be:

{% highlight csharp %}
var gpsPosition = new Vector3d(GPSManager.Instance.position);
var objectPosition = new Vector3d(Position);
var gpsAlt = 0;

var heading = MathHelper.DegreesToRadians(GPSManager.Instance.heading);
// Need to rotate the the the offset to align to the world coords

var t = Quaterniond.FromEulerAngles(0, heading, 0);
var obj = GameObject.CreatePrimitive(PrimitiveType.Cylinder);

var gpsObj = obj.AddComponent<GPSTrackedObject>();
gpsObj.GpsPosition = Position;
gpsObj.GPSManager = GPSManager.Instance;

{% endhighlight %}
