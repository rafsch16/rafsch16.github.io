---
layout: post
title:  "Creating and Deploying a Dashboard"
date:   2021-11-30 12:35:09 +0200
categories: dashboard datascience
---

<html>
<head>
<meta charset="utf-8">
<style>

#img1 {
display: block;
width: 40%;
margin-left: 30%;
margin-right: auto;
}

#img2 {
display: block;
width: 100%;
margin-left: 0%;
margin-right: auto;
}

.custom {
 font-family: Courier;
 color: red;
 font-size:16px;
 }

p1 {
  font-size: 14px;
}
</style>
</head>
<body>

<img src="\images\sky-panorama.jpg" />
<br>
<br>

<p>This blog post explains how to build a data dashboard as a web-app and deploy it on Heroku.</p>

<h3>Overview</h3>

<p>One of the most important aspects in data science is to visualize data. This can be achieved using a data <b>dashboard</b> that provides information at a glance. Ideally, a dashboard is configurable and linked to a database, allowing to select and filter the data. Moreover, the data dashboard can be deployed as a web-app running on a web server for convenience of sharing the data.</p>

<p>Here, I will show how to create a simple dashboard that allows to select and visualize weather forecasts for cities worldwide. The data is requested from the <b>Openweathermap data API</b> using the <b>Requests</b> library and visualized using the <b>Plotly</b> library. The webpage is rendered using <b>Flask</b> and its template engine <b>Jinja</b>. To make development of the front-end easy, this will make use of the <b>Bootstrap</b> and <b>jQuery</b> libraries. Deployment of the dashboard as a web app will then be done using <b>Heroku</b>. In the following sections, a step by step implementation of the dashboard will be presented. I will only show some important code snippets for the understanding, the full code can be found on my <a href="https://github.com/rafsch16/weather-app">github</a> page. Some of the code was reused from the Udacity Nanodegree Data Scientist program. The deployed web-app can be accessed <a href="https://weather-app-rafsch16.herokuapp.com">here</a>.</p>

<h3>Flask</h3>

<p>Flask is a web framework that takes care of all the routing needed to organize a web page. A simple application of Flask could look like this:</p>

{% highlight python %}
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello_world():
    return "<p>Hello, World!</p>"
{% endhighlight %}

<p>The <span class="custom">@app.route()</span> decorator is used to tell Flask what URL will trigger the <span class="custom">hello_world( )</span> function. This function will return the message that will be displayed in a user's browser. The content type of this message is HTML by default.</p>

<p>For this web-app, <span class="custom">@app.route()</span> is used to render a HTML template:</p>

{% highlight python %}
from flask import render_template

@app.route('/', methods=['POST', 'GET'])
@app.route('/index', methods=['POST', 'GET'])
def index():

    [... some code ...]

    # get figures for selected cities
    figures = return_figures(cities_selected)

    # list ids for the html id tag
    ids = ['figure-{}'.format(i) for i, _ in enumerate(figures)]
	
    # convert the plotly figures to JSON for javascript in html template
    figuresJSON = json.dumps(figures, cls=plotly.utils.PlotlyJSONEncoder)

    # render the template
    return render_template('index.html', ids=ids, figuresJSON=figuresJSON,
		all_cities=cities_default,
		cities_selected=cities_selected)
{% endhighlight %}

<p>For <span class="custom">@app.route()</span> we have two triggers, for the URL's <span class="custom">'/'</span> and <span class="custom">'/index'</span>, and we set the argument <span class="custom">methods=['POST', 'GET']</span>. This enables the HTTP methods POST for sending data and GET to request data. By default, a route only answers to GET requests. I will show later how the app makes use of the POST method.</p>

<p>The Flask function <span class="custom">render_template()</span> renders <span class="custom">index.html</span> stored in the templates folder. For sharing data between the back-end and front-end, the figures, the id's of the figures, and the names of the selected cities are included as an argument. Before we have a look at the file <span class="custom">index.html</span>, I will show how the figures are initialized in the function <span class="custom">return_figures()</span>.</p>

<h3>Plotly</h3>

<p>Plotly is a web chart library and provides open source libraries for both JavaScript and Python. The example below shows how to define a graph:</p>

{% highlight python %}
import plotly.graph_objs as go

# creat the garph one
graph_one = []

for i in range(len(city_names)):
    df_one = pd.DataFrame(data_frames[i]['list'])
    x_t = df_one.dt.tolist()
    x_val =  []
    y_val =  []
    for j in range(df_one.main.shape[0]):
        x_val.append(datetime.utcfromtimestamp(x_t[j]).strftime('%Y-%m-%d %H:%M:%S'))
        y_val.append(df_one.main[j]['temp'])
    graph_one.append(
        go.Scatter(
        x = x_val,
        y = y_val,
        mode = 'lines',
        name = city_names[i]
        )
    )

layout_one = dict(
            xaxis = dict(tickformat = '%d %b %H:%M'),
            yaxis = dict(title = {'text': 'Temperature (°C)', 'font': {'size': 16}}),
            images = [dict(source = "/static/img/temperature.png",
                            xref = "paper",
                            yref = "paper",
                            x = 0,
                            y = 1,
                            sizex = 0.3,
                            sizey = 0.3,
                            xanchor = "right",
                            yanchor = "bottom")],
            )
{% endhighlight %}

<p>Important here is that a <span class="custom">go.Scatter()</span> object is appended to the list <span class="custom">graph_one</span> for each city of interest. Other graph objects than Scatter are available in the <span class="custom">graphs_obj</span> package. Next, the layout of the Scatter object is defined using a dictionary. Once all the graphs were defined, they can be appended to a <span class="custom">figures</span> list as a dictionary by including <span class="custom">data</span> and <span class="custom">layout</span> keys. This information will be used later to display the figures with help of Plotly.js.</p>

{% highlight python %}
[...some code...]

figures = []
figures.append(dict(data=graph_one, layout=layout_one))
figures.append(dict(data=graph_two, layout=layout_two))
figures.append(dict(data=graph_three, layout=layout_three))
figures.append(dict(data=graph_four, layout=layout_four))
figures.append(dict(data=graph_five, layout=layout_five))
figures.append(dict(data=graph_six, layout=layout_six))

return figures
{% endhighlight %}

<h3>Requests</h3>

<p>You might have wondered how the data was obtained to define the graphs in the previous paragraph. The data was requested from the Openweathermap API using the Requests library. The code below shows how to perform a basic GET request:</p>

{% highlight python %}
import requests

# assemble url for API request
url = 'http://api.openweathermap.org/data/2.5/forecast?q=' + city + \
        '&exclude=hourly,minutely,alerts'+'&units=metric' +'&appid=' + api_key 

# perform GET request
try:
    r = requests.get(url)
    data = r.json()

except:
    print('GET request failed')
{% endhighlight %}

<p>Here the <span class="custom">get()</span> method is used and the <span class="custom">json()</span> decoder is applied for JSON data.</p>

<h3>Bootstrap and Jinja</h3>

<p>We can now have a look at the <span class="custom">index.html</span> file that serves as a template for the front-end. Here, Bootstrap is used for web development, which eliminates the need to write CSS or JavaScript. Instead, the webpage can be styled in HTML. The Bootstrap library depends on jQuery and we first have to import the script files needed for Bootsrap and Plotly in the <span class="custom">&lt;head&gt;</span> section:</p>

{% highlight html %}
{% raw %}
<head>

<title>Openweathermap Data Dashboard</title>

<!--import script files needed for plotly and bootstrap-->
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css" integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm" crossorigin="anonymous">
<script src="https://code.jquery.com/jquery-3.3.1.min.js" integrity="sha384-tsQFqpEReu7ZLhBV2VZlAu7zcOV+rXbYlF2cqB8txI/8aZajjp4Bqd+V6D5IgvKT" crossorigin="anonymous"></script>	
<script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.12.9/umd/popper.min.js" integrity="sha384-ApNbgh9B+Y1QKtv3Rn7W3mgPxhU9K/ScQsAP7hUibX39j7fakFPskvXusvfa0b4Q" crossorigin="anonymous"></script>
<script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/js/bootstrap.min.js" integrity="sha384-JZR6Spejh4U02d8jOt6vLEHfe/JQGiRRSQQxSfFWpi1MquVdAyjUar5+76PVCmYl" crossorigin="anonymous"></script>
<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>

</head>
{% endraw %}
{% endhighlight %}

<p>Bootstrap uses a grid system to layout and align content. Containers provide a means to center the content, whereas rows are wrappers for columns that contain the content. Column classes indicate the number of columns to use out of the possible 12 per row. In the code below, the grid system is shown for the six figures displayed on the webpage:</p>

{% highlight html %}
{% raw %}
<!--charts-->		
<div id="charts" class="container-fluid">
            
    <!--top two charts-->		
    <div class="row">
        <div class="col-6">
            <div id="{{ids[0]}}"></div>
        </div>
        <div class="col-6">
            <div id="{{ids[1]}}"></div>
        </div>
    </div>

    <!--middle two charts-->		
    <div class="row">
        <div class="col-6">	
            <div id="{{ids[2]}}"></div>
        </div>
        <div class="col-6">
            <div id="{{ids[3]}}"></div>
        </div>
    </div>
    
    <!--bottom two charts-->		
    <div class="row">
        <div class="col-6">	
            <div id="{{ids[4]}}"></div>
        </div>
        <div class="col-6">
            <div id="{{ids[5]}}"></div>
        </div>
    </div>
</div>
{% endraw %}
{% endhighlight %}

<p>The <span class="custom">.container-fluid</span> class spans the entire width of the viewport. In each row two figures are displayed, with the <span class="custom">.col-6</span> class indicating each figure to span six out of twelve possible columns.</p>

<p>For the figures we are using <span class="custom">&lt;div&gt;</span> elements with id's specified as <span class="custom">id="&#123;&#123;ids[i]&#125;&#125;"</span>. Here, the Jinja squiggly bracket syntax <span class="custom">&#123;&#123;...&#125;&#125;</span> is used to access data that was passed to the front-end using the <span class="custom">render_template( )</span> method.</p>

<p>Bootstrap can also be used to implement forms for user input. For the web-app I used a checkbox form to filter from a default list of cities. The code below shows how the <span class="custom">dropdown</span> and <span class="custom">form-check</span> classes are used to implement a checkbox form:</p>

{% highlight html %}
{% raw %}
<!-- dropdown menu for filter -->
<div class="dropdown">
    <form class="px-4 py-3" role="form" method="post" action="/" id="form-filter">
        {% for city in all_cities %}
            <div class="form-check">
                <!-- Check the city filter boxes for all cities submitted from the form -->
                {% if city in cities_selected %}
                    <input class="form-check-input city-check" type="checkbox" name="{{ city }}" id="defaultCheck1-{{ city }}" checked>
                {% else %}
                    <input class="form-check-input city-check" type="checkbox" name="{{ city }}" id="defaultCheck1-{{ city }}">							
                {% endif %}
                <label class="form-check-label" for="defaultCheck1-{{city}}">{{city}}</label>
            </div>
        {% endfor %}
        <button id="city_selector" type="submit" class="btn btn-primary my-1">Submit</button>
    </form>
</div>
{% endraw %}
{% endhighlight %}

<p>In the <span class="custom">&lt;form&gt;</span> element the attribute <span class="custom">method="post"</span> is used, in the <span class="custom">&lt;input&gt;</span> element the attribute <span class="custom">type="checkbox"</span> is used. Notice also the <span class="custom">&lt;button&gt;</span> element to submit the information to the back-end Flask scripts. The Jinja squiggly bracket syntax <span class="custom">&#123;&#123;city&#125;&#125;</span> is used to access the city names in the form. Similarly, the Jinja syntax <span class="custom">&#123;%...%&#125;</span> is used for control flow statements.</p>

<p>The second form in the web-app can be used to add new cities. Therefore the <span class="custom">form-floating</span> class is used to implement a text form:</p>

{% highlight html %}
{% raw %}
<!-- text input for new cities -->
<form class="form-vertical" role="form" method="post" action="/" id="form-add">
        <div class="form-floating">
            <input class="form-control" type="text" name="new_city" placeholder="Add a new city here" id="floatingTextarea"></textarea>
            <label for="floatingTextarea">e.g., 'Zürich,CH' or 'London,GB' and press Submit (see <a href="https://en.wikipedia.org/wiki/List_of_ISO_3166_country_codes">ISO 3166 country codes</a>)</label>
        </div>
    <input type="submit" class="btn btn-default" value="Submit"/>
</form>
{% endraw %}
{% endhighlight %}

<p>Again, in the <span class="custom">&lt;form&gt;</span> element the attribute <span class="custom">method="post"</span> is used, whereas in the <span class="custom">&lt;input&gt;</span> element the attribute <span class="custom">type="text"</span> is used.</p>

<p>In the footer section we then use Plotly.js to visualize the figures:</p>

{% highlight html %}
{% raw %}
<body>

[...some code...]

<!--footer section-->				
<div id="footer" class="container"></div>

</body>


<footer>
    <script type="text/javascript">
    	// plots the figure by id using Plotly
    	// id much match the div id above in the html
    	var figures = {{figuresJSON | safe}};
		var ids = {{ids | safe}};
		for(var i in figures) {
			Plotly.plot(ids[i],
				figures[i].data,
				figures[i].layout || {});
		};

    [...some code...]

    </script>

</footer>
{% endraw %}
{% endhighlight %}

<p>Here, using Jinja and JavaScript syntax, the figures are plotted by the id's that match the <span class="custom">&lt;div&gt;</span> id's in the HTML code.</p>

<h3>Flask again</h3>

<p>For receiving and interpreting the submitted information from the forms, the Flask <span class="custom">request</span> object is used:</p>

{% highlight python %}
if (request.method == 'POST') and request.form:

    # retrieve user input
    req=request.form

    req_form = list(req.keys())
    
    cities_selected = []

    for city in req_form:
        cities_selected.append(city)
{% endhighlight %}

<p>The transition method is validated with the method attribute, i.e., <span class="custom">request.method == 'POST'</span>, and the form parameters are then obtained from <span class="custom">request.form</span>. As the parameters refer to the cities from the checkbox, we can append them to the selected cities for updating the figures. A similar code is also used for the text form.</p>

<p>The above code closes the cycle of routing at the back-end. Flask initializes the web-page using the <span class="custom">@app.route()</span> method and keeps listening for requests from the user input forms. The information from the forms are then used for sending updated figures to the front-end.</p>

<h3>Running the web-app</h3>

<p>To understand how to run the web-app, it is important to know the structure of the web-app. The file tree is shown in the figure below.</p>

<div id="img1">
    <img src="\images\file_tree.png" />
</div>

<p1><b>Figure 1.</b> File tree of the web-app.</p1>
<br>
<br>
<p>For running the web-app from the CMD, the following commands are used:</p>

{% highlight shell %}
> set FLASK_APP=openweathermap
> flask run
{% endhighlight %}

<p>The syntax is slightly different for Bash and Powershell usage. Flask will run the <span class="custom">openweathermap.py</span> script first, which is located in the root folder. The content of this script is the following: </p>

{% highlight python %}
from weatherapp import app
{% endhighlight %}

<p>Here, <span class="custom">weatherapp</span> is a package containing an <span class="custom">__init__.py</span> file with the initialization code. The content of this file is:</p>

{% highlight python %}
from flask import Flask

app = Flask(__name__)

from weatherapp import routes
{% endhighlight %}

<p>Thus with the command <span class="custom">flask run</span>, the <span class="custom">app</span> instance is created and executed. At the same time the <span class="custom">routes.py</span> script is executed, which contains the routing with the <span class="custom">@app.route()</span> method.</p>

<h3>Deploying on Heroku</h3>

<p>For deploying the web-app on a web server, we need some additional configuration files that were not required for running the app locally. Those files are located in the root folder. First, <span class="custom">requirements.txt</span> lists all the packages needed for running the app, <span class="custom">runtime.txt</span> contains the python version, and <span class="custom">Procfile</span> specifies the commands that are executed by the app on startup. The content of the <span class="custom">Procfile</span> is:</p> 

{% highlight python %}
web: gunicorn openweathermap:app
{% endhighlight %}

<p>The <span class="custom">web</span> process is required for web servers on Heroku, <span class="custom">gunicorn</span> starts the WSGI server used for deployment (replacing <span class="custom">flask</span> that was used locally), and <span class="custom">openweathermap:app</span> is the command to be executed.</p> 

<p>For deploying the app on Heroku, we need to set up a functioning git repository. From the root of the local repository, we can then create an app on Heroku, which prepares Heroku to receive the source code:</p> 

{% highlight shell %}
heroku create my-app-name --buildpack heroku/python
{% endhighlight %}

<p>Next, we can add a remote to the local repository:</p> 

{% highlight shell %}
heroku git:remote -a my-app-name
{% endhighlight %}

<p>To deploy the app to Heroku, the code is pushed from the local repository master or main branch to the heroku remote using the git command:</p> 

{% highlight shell %}
git push heroku main
{% endhighlight %}

<p>Now the app can be visited at the URL generated by its app name or using the command <span class="custom">heroku open</span>.</p> 

<p>And this is how the web-app looks:</p>

<div id="img2">
    <img src="\images\web_app.png" />
</div>

<p1><b>Figure 2.</b> Screenshot of the weather forcast web-app.</p1>
<br>
<br>
<h3>References</h3><a name="references"></a>
<p id="refs">
[1] <a href="https://openweathermap.org">OpenWeatherMap: Сurrent weather and forecast</a>
</p>
<p>
[2] <a href="https://flask.palletsprojects.com/en/2.0.x/">Welcome to Flask — Flask Documentation (2.0.x)</a>
</p>
<p>
[3] <a href="https://plotly.com">Plotly: The front end for ML and data science models</a>
</p>
<p>
[4] <a href="https://getbootstrap.com">Bootstrap: The most popular HTML, CSS, and JS library in the world</a>
</p>
<p>
[5] <a href="https://www.heroku.com/home">Heroku: Cloud Application Platform</a>
</p>
<p>
[5] <a href="https://www.udacity.com">Udacity: Learn the Latest Tech Skills</a>
</p>


</body>
</html>