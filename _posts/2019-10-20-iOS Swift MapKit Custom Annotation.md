---
layout: post
title: iOS Swift MapKit Custom Annotation
tags: Project
categories: Tech
cover: "upload/blog_masonry_04.jpg"
---

<br>
Outline

We will create a point annotation object and assigning a custom image name with the CustomPointAnnotation class.

We will subclass the MKPointAnnotation to set image and assign it on the delegate protocol method viewForAnnotation.

We will add an annotation view to the map after setting the coordinate of the point annotation with a title and a subtitle.

We will implement the viewForAnnotation method which is an MKMapViewDelegate protocol method which gets called for pins to display on the map. viewForAnnotation protocol method is the best place to customise the pin view and assign a custom image to it.

We will dequeue and return a reusable annotation for the given identifier and cast the annotation to our custom CustomPointAnnotation class in order to access the image name of the pin.

We will create a new image set in Assets.xcassets and place image@3x.png and image@2x.png accordingly.
<br>


{% highlight swift %}
CustomPointAnnotation.swift

import UIKit
import MapKit

class CustomPointAnnotation: MKPointAnnotation {
    var pinCustomImageName:String!
}
{% endhighlight %}

{% highlight swift %}
ViewController.swift

import UIKit
import MapKit

class ViewController: UIViewController, MKMapViewDelegate,  CLLocationManagerDelegate {


    @IBOutlet weak var pokemonMap: MKMapView!
    let locationManager = CLLocationManager()
    var pointAnnotation:CustomPointAnnotation!
    var pinAnnotationView:MKPinAnnotationView!

    override func viewDidLoad() {
        super.viewDidLoad()

        //Mark: - Authorization
        locationManager.delegate = self
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        locationManager.requestAlwaysAuthorization()
        locationManager.startUpdatingLocation()

        pokemonMap.delegate = self
        pokemonMap.mapType = MKMapType.Standard
        pokemonMap.showsUserLocation = true

    }

    func locationManager(manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        let location = CLLocationCoordinate2D(latitude: 35.689949, longitude: 139.697576)
        let center = location
        let region = MKCoordinateRegionMake(center, MKCoordinateSpan(latitudeDelta: 0.025, longitudeDelta: 0.025))
        pokemonMap.setRegion(region, animated: true)

        pointAnnotation = CustomPointAnnotation()
        pointAnnotation.pinCustomImageName = "Pokemon Pin"
        pointAnnotation.coordinate = location
        pointAnnotation.title = "POKéSTOP"
        pointAnnotation.subtitle = "Pick up some Poké Balls"

        pinAnnotationView = MKPinAnnotationView(annotation: pointAnnotation, reuseIdentifier: "pin")
        pokemonMap.addAnnotation(pinAnnotationView.annotation!)
    }

    func locationManager(manager: CLLocationManager, didFailWithError error: NSError) {
        print(error.localizedDescription)
    }

    //MARK: - Custom Annotation
    func mapView(mapView: MKMapView, viewForAnnotation annotation: MKAnnotation) -> MKAnnotationView? {
        let reuseIdentifier = "pin"
        var annotationView = mapView.dequeueReusableAnnotationViewWithIdentifier(reuseIdentifier)

        if annotationView == nil {
            annotationView = MKAnnotationView(annotation: annotation, reuseIdentifier: reuseIdentifier)
            annotationView?.canShowCallout = true
        } else {
            annotationView?.annotation = annotation
        }

        let customPointAnnotation = annotation as! CustomPointAnnotation
        annotationView?.image = UIImage(named: customPointAnnotation.pinCustomImageName)

        return annotationView
    }
}

{% endhighlight %}

[[Ref.] StackOverflow - Swift MapKit Custom Annotation](https://stackoverflow.com/questions/38274115/ios-swift-mapkit-custom-annotation)



