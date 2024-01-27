# Spring Gradle Rest Services

Lets write a connection to an external API. We will be using the [XKCD API](https://xkcd.com/json.html) since it is easy to incorporate and more importantly free.

Lets start with creating the file `XKCDController` within out `/controller` folder. Within it we will create a GET mapping request so we can connect from our browser or postman to our backend spring application and then on to the XKCD API. For the return on our API we will be using an object. Lets call it XkcdComic.java and we will create it in our `/domain` folder.

```java
// XKCDController
@RestController
public class XKCDController {

    @GetMapping("/xkcd")
    public XkcdComic xkcdComic(){
        
    }
}
```
Before we create our object we need to take a look at our API we are integrating with. In this case if you go to the documenation you will see two endpoints. One is for the current and one is for the past, or specific, comic number. Lets start with the current. If we click on its link [https://xkcd.com/info.0.json](https://xkcd.com/info.0.json) we will see the output in this format for the keys, values removed:
```json
{
    "month": "",
    "num": 1,
    "link": "",
    "year": "",
    "news": "",
    "safe_title": "",
    "transcript": "",
    "alt": "",
    "img": "",
    "title": "",
    "day": ""
}
```
Knowing that these are the keys we need to create an object that matches these keys and types. We create the object and declare the fields as private like so.  
```java
public class XkcdComic {
    private String month;
    private int num;
    private String link;
    private String year;
    private String news;
    private String safe_title;
    private String transcript;
    private String alt;
    private String img;
    private String title;
    private String day;
}
```
The fields are encapsulated, meaning they are declared as private and accessed through public getter and setter methods. This ensures proper encapsulation and allows for better control over the object's state. The get or set methods are added below the fields for readability and each field will contain its own get and set method:
```java
public getMonth(){
    return month;
}
public setMonth(String mon){
    month = mon;
}
```
You can see how this could quickly grow out of hand as the object gains more and more fields. To control this we will use `lombok`. It will automatically create our getters and setters for all our fields and we will not have to write any. We simply modify the object we have now to include two annotations:
```java
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class XkcdComic {
    private String month;
    private int num;
    private String link;
    private String year;
    private String news;
    private String safe_title;
    private String transcript;
    private String alt;
    private String img;
    private String title;
    private String day;
}
```

Now that we have the response object lets go fill in our container. We will be using the `RestTemplate` class to make our API call to the XKCD api. Within the controller method we create a new RestTemplate variable and then use its `getForObject` method. This method takes two arguments. The first is a string that contains the URL we are trying to get information from and the second is a java class for the response. This class will be the same one we just created. Lets write the code out:
```java
@RestController
public class XKCDController {

    @GetMapping("/xkcd")
    public XkcdComic xkcdComic(){
        RestTemplate restTemplate = new RestTemplate();
        XkcdComic result = restTemplate.getForObject("https://xkcd.com/info.0.json", XkcdComic.class);
        return result;
    }
}
```

Now if we start our application using `gradle bootRun` and navigate to the `/xkcd` endpoint you should be able to see the data from the XKCD api.

Now lets go ahead and connect to the second API available for XKCD. This one requires an additional step. It takes a param for the comic number. We can look up past comics using that. We may want that as well and for it we need to to be able to pass the variable from the browser to the backend. Lets start by copying the GET for the last comic we created and make some modifications.

Lets start by adding a path variable to the URL named comicNumber and adding the corresponding information to the method class. Now we have to also modify the URL in the getForObject method: 

```java
@RestController
public class XKCDController {

    @GetMapping("/xkcd")
    public XkcdComic xkcdComic(){
        RestTemplate restTemplate = new RestTemplate();
        XkcdComic result = restTemplate.getForObject("https://xkcd.com/info.0.json", XkcdComic.class);
        return result;
    }

    @GetMapping("/xkcd/{comicNumber}")
    public XkcdComic xkcdComicPast(@PathVariable String comicNumber){
        RestTemplate restTemplate = new RestTemplate();
        XkcdComic result = restTemplate.getForObject("https://xkcd.com/" + comicNumber + "/info.0.json", XkcdComic.class);
        return result;
    }
}
```

Before we start this we have an inherent flaw in that our root URL is the same for the GET url for the current and past comic. Since we have the same root URL we need to declare it at a class method. Lets modify that like so:
```java
@RestController
@RequestMapping("/xkcd")
public class XKCDController {
    ...
}
```
This will create a root URL for the class that will be prepended to our other methods URLs. So lets modify our two urls to `/current` and `/past/{comicNumber}` 

Now we can restart our app and navigate to `/xkcd/current` and we should still see the data we had before. Now if we change and go to `/xkcd/past/1234` we should be pulling comic 1234, change that to any other number less than the currents number and we should see them all.
