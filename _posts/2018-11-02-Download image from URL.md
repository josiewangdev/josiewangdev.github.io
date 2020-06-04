---
layout: post
title: "Download image from URL"
tags: [iOS]
categories: [Tech]
cover: "upload/blog_masonry_05.jpg"
---

* Synchronously

{% highlight swift %}
if let filePath = Bundle.main.path(forResource: "imageName", ofType: "jpg"), let image = UIImage(contentsOfFile: filePath) {
    imageView.contentMode = .scaleAspectFit
    imageView.image = image
}
{% endhighlight %}

* Asynchronously
* Create a method with a completion handler to get the image data from your url

{% highlight swift %}
func getData(from url: URL, completion: @escaping (Data?, URLResponse?, Error?) -> ()) {
    URLSession.shared.dataTask(with: url, completionHandler: completion).resume()
}
{% endhighlight %}

* Create a method to download the image (start the task)

{% highlight swift %}
func downloadImage(from url: URL) {
    print("Download Started")
    getData(from: url) { data, response, error in
        guard let data = data, error == nil else { return }
        print(response?.suggestedFilename ?? url.lastPathComponent)
        print("Download Finished")
        DispatchQueue.main.async() {
            self.imageView.image = UIImage(data: data)
        }
    }
}
{% endhighlight %}

* Usage

{% highlight swift %}
override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view, typically from a nib.
    print("Begin of code")
    let url = URL(string: "https://cdn.arstechnica.net/wp-content/uploads/2018/06/macOS-Mojave-Dynamic-Wallpaper-transition.jpg")! 
    downloadImage(from: url)
    print("End of code. The image will continue downloading in the background and it will be loaded when it ends.")
}
{% endhighlight %}

* Extension:

{% highlight swift %}
extension UIImageView {
    func downloaded(from url: URL, contentMode mode: UIViewContentMode = .scaleAspectFit) {
        // for swift 4.2 syntax just use ===> mode: UIView.ContentMode
        contentMode = mode
        URLSession.shared.dataTask(with: url) { data, response, error in
            guard
                let httpURLResponse = response as? HTTPURLResponse, httpURLResponse.statusCode == 200,
                let mimeType = response?.mimeType, mimeType.hasPrefix("image"),
                let data = data, error == nil,
                let image = UIImage(data: data)
                else { return }
            DispatchQueue.main.async() {
                self.image = image
            }
        }.resume()
    }
    func downloaded(from link: String, contentMode mode: UIViewContentMode = .scaleAspectFit) {  // for swift 4.2 syntax just use ===> mode: UIView.ContentMode
        guard let url = URL(string: link) else { return }
        downloaded(from: url, contentMode: mode)
    }
}
{% endhighlight %}

* Usage

{% highlight swift %}
imageView.downloaded(from: "https:/....jpg")
{% endhighlight %}
  
