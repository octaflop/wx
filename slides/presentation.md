# Time Series Databases in Python: Building Data Pipelines and Weather Visualizations

This presentation explores the world of time series databases with Python, focusing on how to collect, store, and visualize weather data effectively. We'll compare different open-source time series database options, dive deep into implementation with Python, and build a complete weather dashboard as our end product.

## Introduction to Time Series Data

Time series data consists of measurements or observations collected sequentially over time. This type of data is ubiquitous in our digital world and appears in many contexts:

- Weather measurements (temperature, humidity, pressure)
- Server and application metrics
- Financial markets data
- IoT sensor readings
- Social media activity patterns

What makes time series data special is that time is the primary axis, with data typically being append-only and requiring specific query patterns like time-based aggregations[19]. Standard relational databases often struggle when data volumes reach billions of rows, which is common for time series applications[20].

### The Need for Specialized Time Series Databases

Traditional databases were not designed with time series workloads in mind. Time series databases offer several key advantages:

- **Performance optimizations**: Specialized indexing and compression for time-based data
- **Time-based functions**: Built-in aggregations, rolling windows, and downsampling
- **Scalability**: Designed for high write throughput and efficient storage
- **Data lifecycle management**: Automatic retention policies and data tiering

As search results show, "Time Series Databases is like a super Dragon. They are built for Big Time Series Data and outperform Data Warehouses as well as relational databases"[19].

## Key Features to Look for in a Time Series Database

When evaluating time series databases for Python projects, consider these important factors:

### Performance Characteristics

Performance benchmarks vary widely between databases. According to ClickBench comparisons, relative runtimes range significantly with some databases performing twice as fast as others[2]. Key metrics include:

- Write throughput (points per second)
- Query response time
- Compression efficiency
- Memory utilization

### Python Integration Quality

For our Python meetup audience, the quality of Python libraries is crucial:

- Native client library support
- ORM compatibility
- Query interface (SQL or custom)
- Documentation and examples
- Community support

### Visualization Capabilities

Since our end goal is to build an impressive dashboard, visualization support is paramount:

- Integration with dashboarding tools
- Data export formats
- Real-time updating
- Interactive query capabilities

## Overview of Open Source Time Series Databases

Let's examine the most popular open-source time series database options and their Python support.

### InfluxDB

InfluxDB is a purpose-built time series database designed for high-performance time series applications.

**Key Features:**
- Purpose-built for time series data with no external dependencies
- Millions of downloads, ranked #1 time series database according to DB Engines[1]
- Excellent Python client library with comprehensive documentation
- SQL query support alongside Flux query language
- High performance at scale: "Manage millions of time series data points per second without limits"[1]

**Python Integration:**
The influxdb-client package provides a clean, Pythonic interface for all InfluxDB operations:

```python
import influxdb_client
from influxdb_client.client.write_api import SYNCHRONOUS

client = influxdb_client.InfluxDBClient(
    url="http://localhost:8086",
    token="my-token",
    org="my-org"
)
```

### TimescaleDB

TimescaleDB extends PostgreSQL with time series capabilities, offering the familiarity of SQL with specialized time-based optimizations.

**Key Features:**
- Built on PostgreSQL, leveraging its ecosystem
- Full SQL support with additional time-based functions
- Automatic time partitioning through hypertables
- SQLAlchemy integration for Python developers

**Python Integration:**
TimescaleDB can be accessed through standard PostgreSQL drivers like psycopg2, and there's also a specialized SQLAlchemy dialect[4]:

```python
from sqlalchemy import create_engine
from sqlalchemy import Table, Column, Integer, String, DateTime

engine = create_engine('timescaledb://user:password@host:port/database')
```

### QuestDB

QuestDB positions itself as a high-performance SQL database designed specifically for time series.

**Key Features:**
- High-performance SQL database optimized for time series
- Focuses on query speed with fast HTTP/CSV interface
- Pandas-friendly results format
- Designed for analytics workloads

**Python Integration:**
QuestDB offers a Python client that returns query results as Pandas dataframes[7]:

```python
from questdb_query import pandas_query, Endpoint

endpoint = Endpoint("http://localhost:9000")
df = pandas_query('select * from weather limit 1000', endpoint=endpoint)
```

### Prometheus

While primarily a monitoring system, Prometheus includes a powerful time series database component.

**Key Features:**
- Monitoring-focused time series database
- Pull-based metrics collection model
- Built-in alerting system
- Strong Grafana integration

**Python Integration:**
The Prometheus Python client focuses on metrics collection rather than querying[5]:

```python
from prometheus_client import start_http_server, Summary

REQUEST_TIME = Summary('request_processing_seconds', 'Time spent processing request')

@REQUEST_TIME.time()
def process_request(t):
    # Processing logic here
    pass
```

### OpenTSDB

OpenTSDB is designed for distributed storage and querying of time series data.

**Key Features:**
- Built to run on top of HBase for scalability
- Strong focus on monitoring use cases
- Telnet and HTTP APIs
- Grafana integration

**Python Integration:**
OpenTSDB can be queried via its HTTP API using standard Python libraries.

## Time Series Database Benchmark Comparisons

Performance is a critical factor when selecting a time series database. Recent benchmarks show significant performance variations:

**ClickBench Relative Runtime (lower is better)**[2]:
- Umbra: ×1.60
- ClickHouse: ×1.75
- DuckDB: ×2.19
- QuestDB: ×2.62

**Taxi Ride Benchmarks (normalized, lower is better)**[2]:
- kdb+/q: 1.0
- ClickHouse: 2.3
- DuckDB: 2.8
- Hydrolix: 3.7

## Why We're Choosing InfluxDB for This Project

After evaluating the options, we'll focus on InfluxDB for our weather visualization project for several reasons:

1. **Excellent Python integration**: Well-documented client library with intuitive API[11]
2. **Strong visualization capabilities**: Good integration with various visualization tools
3. **Purpose-built for time series**: Optimized storage and query performance[1]
4. **Active community**: Large user base and extensive documentation
5. **Balance of performance and usability**: Competitive performance with developer-friendly interface

## Deep Dive: InfluxDB Concepts and Architecture

Before we start coding, let's understand InfluxDB's core concepts:

### Core Concepts

- **Bucket**: Container for time series data (similar to a database)
- **Measurement**: Collection of related time series (similar to a table)
- **Tag**: Indexed metadata for efficient filtering (like location, sensor_id)
- **Field**: The actual measurements/values being stored (like temperature, humidity)
- **Timestamp**: When the measurement was recorded

### Installing and Setting Up InfluxDB

InfluxDB can be installed locally or run in Docker:

```bash
# Using the official script
curl -O https://www.influxdata.com/d/install_influxdb3.sh
sh install_influxdb3.sh

# Or via Docker
docker pull influxdb:latest
docker run -d -p 8086:8086 --name influxdb influxdb
```

### Python Client Library

Install the InfluxDB Python client:

```bash
pip install influxdb-client
```

Basic client setup:

```python
import influxdb_client
from influxdb_client.client.write_api import SYNCHRONOUS

# Configuration
bucket = "weather_data"
org = "python_meetup"
token = "your-token"  # Generate this in the InfluxDB UI
url = "http://localhost:8086"

# Create client
client = influxdb_client.InfluxDBClient(
    url=url,
    token=token,
    org=org
)
```

## Writing Data to InfluxDB

The Python client makes it easy to write data to InfluxDB[11]:

```python
# Create write API instance
write_api = client.write_api(write_options=SYNCHRONOUS)

# Create a data point
from datetime import datetime
p = influxdb_client.Point("weather")\
    .tag("location", "Salt Lake City")\
    .field("temperature", 25.3)\
    .field("humidity", 62.8)\
    .time(datetime.utcnow())

# Write the point to InfluxDB
write_api.write(bucket=bucket, org=org, record=p)
```

You can also write multiple points in batches for better performance:

```python
points = []
for i in range(10):
    point = influxdb_client.Point("weather")\
        .tag("location", "Salt Lake City")\
        .field("temperature", 20 + i * 0.5)\
        .field("humidity", 60 - i)\
        .time(datetime.utcnow())
    points.append(point)

write_api.write(bucket=bucket, org=org, record=points)
```

## Querying Data from InfluxDB

InfluxDB supports both Flux (its purpose-built query language) and SQL for querying data[11]:

```python
# Create query API instance
query_api = client.query_api()

# Flux query example
query = '''
from(bucket:"weather_data")
  |> range(start: -1d)
  |> filter(fn:(r) => r._measurement == "weather")
  |> filter(fn:(r) => r.location == "Salt Lake City")
  |> filter(fn:(r) => r._field == "temperature")
'''

# Execute query
result = query_api.query(org=org, query=query)

# Process results
for table in result:
    for record in table.records:
        print(f"{record.get_time()}: {record.get_value()}")
```

For data visualization, it's often more convenient to get results as a Pandas DataFrame:

```python
# Query to DataFrame
df = query_api.query_data_frame(query=query)
print(df.head())
```

## Weather Data Collection

Now that we understand InfluxDB basics, let's build a weather data collection system.

### Weather API Considerations

While our initial plan was to use Weather Underground, it has discontinued free API keys[14]. Instead, we'll use alternatives like OpenWeatherMap or the National Weather Service API, which remain free and accessible.

### Weather Data Collection Script

Here's a complete script to fetch weather data and store it in InfluxDB:

```python
import influxdb_client
from influxdb_client.client.write_api import SYNCHRONOUS
import requests
import time
from datetime import datetime

# InfluxDB configuration
bucket = "weather_data"
org = "python_meetup"
token = "your-token"
url = "http://localhost:8086"

# Weather API configuration (using OpenWeatherMap as example)
API_KEY = "your-api-key"
CITY = "Salt Lake City"
WEATHER_URL = f"https://api.openweathermap.org/data/2.5/weather?q={CITY}&appid={API_KEY}&units=metric"

# Set up InfluxDB client
client = influxdb_client.InfluxDBClient(
    url=url,
    token=token,
    org=org
)
write_api = client.write_api(write_options=SYNCHRONOUS)

def get_weather_data():
    """Fetch current weather data from OpenWeatherMap API"""
    response = requests.get(WEATHER_URL)
    if response.status_code != 200:
        raise Exception(f"API error: {response.status_code}")
    
    data = response.json()
    
    return {
        "location": CITY,
        "temperature": data["main"]["temp"],
        "humidity": data["main"]["humidity"],
        "pressure": data["main"]["pressure"],
        "wind_speed": data["wind"]["speed"],
        "conditions": data["weather"][0]["main"]
    }

def write_to_influxdb(weather_data):
    """Write weather data to InfluxDB"""
    point = influxdb_client.Point("weather")\
        .tag("location", weather_data["location"])\
        .tag("conditions", weather_data["conditions"])\
        .field("temperature", float(weather_data["temperature"]))\
        .field("humidity", float(weather_data["humidity"]))\
        .field("pressure", float(weather_data["pressure"]))\
        .field("wind_speed", float(weather_data["wind_speed"]))\
        .time(datetime.utcnow())
    
    write_api.write(bucket=bucket, org=org, record=point)
    print(f"Data written to InfluxDB at {datetime.now()}")

# Main data collection loop
def main():
    print(f"Starting weather data collection for {CITY}")
    while True:
        try:
            weather_data = get_weather_data()
            write_to_influxdb(weather_data)
        except Exception as e:
            print(f"Error: {e}")
        
        # Wait for 5 minutes before next collection
        time.sleep(300)

if __name__ == "__main__":
    main()
```

## Building a Weather Dashboard with NiceGUI

### Creating a Weather Dashboard


# NiceGUI

```python
from datetime import datetime, timedelta
import influxdb_client
import pandas as pd
import plotly.express as px
from nicegui import ui, app

# InfluxDB configuration
bucket = "weather_data"
org = "python_meetup"
token = "your-token"
url = "http://localhost:8086"

@ui.refreshable
def create_dashboard(time_range: str = '-1d'):
    # Create InfluxDB client
    client = influxdb_client.InfluxDBClient(url=url, token=token, org=org)
    query_api = client.query_api()

    # Build Flux query
    query = f'''
    from(bucket:"{bucket}")
      |> range(start: {time_range})
      |> filter(fn:(r) => r._measurement == "weather")
      |> filter(fn:(r) => r.location == "Salt Lake City")
    '''
    
    # Execute query and process results
    result = query_api.query_data_frame(query=query)
    
    if result.empty:
        ui.notify("No data available for selected time range", type='warning')
        return

    # Process data
    df_pivot = result.pivot_table(
        index="_time", 
        columns="_field", 
        values="_value"
    ).reset_index()

    # Current conditions row
    with ui.row().classes('w-full items-stretch'):
        with ui.card().classes('w-full'):
            with ui.row().classes('items-center justify-between'):
                ui.label("Current Weather Conditions").classes('text-2xl')
                ui.button(on_click=create_dashboard.refresh, icon='refresh') \
                    .classes('self-end')
                
            with ui.grid(columns=4).classes('w-full gap-4'):
                latest = df_pivot.iloc[-1]
                with ui.card().classes('text-center p-4'):
                    ui.label("Temperature").classes('text-lg')
                    ui.label(f"{latest['temperature']:.1f} °C") \
                        .classes('text-4xl font-bold text-blue-600')
                
                with ui.card().classes('text-center p-4'):
                    ui.label("Humidity").classes('text-lg')
                    ui.label(f"{latest['humidity']:.1f} %") \
                        .classes('text-4xl font-bold text-green-600')
                
                with ui.card().classes('text-center p-4'):
                    ui.label("Pressure").classes('text-lg')
                    ui.label(f"{latest['pressure']:.1f} hPa") \
                        .classes('text-4xl font-bold text-purple-600')
                
                with ui.card().classes('text-center p-4'):
                    ui.label("Wind Speed").classes('text-lg')
                    ui.label(f"{latest['wind_speed']:.1f} m/s") \
                        .classes('text-4xl font-bold text-orange-600')

    # Time series charts
    with ui.row().classes('w-full'):
        with ui.card().classes('w-full p-4'):
            ui.label("Temperature Over Time").classes('text-xl mb-4')
            fig_temp = px.line(
                df_pivot, 
                x="_time", 
                y="temperature",
                labels={"_time": "Time", "temperature": "Temperature (°C)"},
                line_shape="spline"
            )
            ui.plotly(fig_temp).classes('w-full h-96')

    with ui.row().classes('w-full gap-4'):
        with ui.card().classes('flex-1 p-4'):
            ui.label("Humidity Over Time").classes('text-xl mb-4')
            fig_hum = px.line(
                df_pivot, 
                x="_time", 
                y="humidity",
                labels={"_time": "Time", "humidity": "Humidity (%)"},
                line_shape="spline"
            )
            ui.plotly(fig_hum).classes('w-full h-96')
        
        with ui.card().classes('flex-1 p-4'):
            ui.label("Pressure Over Time").classes('text-xl mb-4')
            fig_press = px.line(
                df_pivot, 
                x="_time", 
                y="pressure",
                labels={"_time": "Time", "pressure": "Pressure (hPa)"},
                line_shape="spline"
            )
            ui.plotly(fig_press).classes('w-full h-96')

    # Data table and export
    with ui.card().classes('w-full p-4'):
        with ui.row().classes('items-center justify-between mb-4'):
            ui.label("Recent Readings").classes('text-xl')
            
            # Download button
            csv = df_pivot.to_csv(index=False)
            ui.download(
                label="Export CSV",
                content=csv,
                filename=f"weather_data_{datetime.now().date()}.csv"
            )
        
        # Interactive data table
        grid = ui.aggrid.from_pandas(df_pivot.tail(10)).classes('max-h-96')
        grid.options.pagination = True
        grid.options.paginationPageSize = 10
        grid.update()

# Time range selection drawer
with ui.left_drawer(fixed=True).classes('bg-gray-100 p-4 w-64'):
    ui.label("Time Range").classes('text-xl mb-4')
    
    time_ranges = {
        "Last hour": "-1h",
        "Last 12 hours": "-12h",
        "Last day": "-1d",
        "Last week": "-1w",
        "Last month": "-30d"
    }
    
    for label, value in time_ranges.items():
        ui.button(
            label, 
            on_click=lambda _, v=value: create_dashboard.refresh(v)
        ).classes('w-full mb-2')

# Main page layout
@ui.page('/')
def main_page():
    with ui.header(elevated=True).style('background-color: #3874c8').classes('items-center'):
        ui.label("Weather Dashboard").classes('text-2xl')
    
    create_dashboard()
    
    with ui.footer().style('background-color: #3874c8'):
        ui.label("Created for Python Meetup - Salt Lake City").classes('text-white')

# Auto-refresh every 5 minutes
ui.timer(300, lambda: create_dashboard.refresh())

# Run the app
ui.run(
    title="Weather Dashboard",
    favicon="🌤️",
    dark=True,
    reload=False
)
```

## Conclusion: Bringing It All Together

In this presentation, we've explored the world of time series databases in Python, with a specific focus on building weather visualizations. We've learned:

1. **Why specialized time series databases matter** for efficiently storing and querying time-based data[19][20]
2. **Key features to consider** when selecting a time series database for Python projects
3. **Comparing popular open-source options** including InfluxDB, TimescaleDB, QuestDB, and others[2][16]
4. **Building a complete data pipeline** for collecting weather data with Python
5. **Creating interactive visualizations** using nicegui for real-time weather monitoring

The combination of InfluxDB for storage and nicegui for visualization provides an impressive, open-source stack for building time series applications entirely in Python. This approach offers excellent performance, good developer experience, and impressive visualization capabilities - perfect for showcasing at our Python meetup.

## Next Steps

To take this project further, consider:

1. Adding forecasting models using scikit-learn or Prophet
2. Implementing anomaly detection for unusual weather patterns
3. Expanding to multiple locations for comparison
4. Adding alerting capabilities for extreme weather conditions
5. Deploying the solution to a cloud provider for continuous data collection

The complete code for this presentation is available on GitHub, including the data collection script, dashboard application, and slide deck.

Citations:
[1] https://www.influxdata.com
[2] https://www.timestored.com/data/time-series-database-benchmarks
[3] https://www.youtube.com/watch?v=49hKs_H5Xf0
[4] https://pypi.org/project/sqlalchemy-timescaledb/
[5] https://prometheus.github.io/client_python/
[6] https://pypi.org/project/WunderWeather/
[7] https://github.com/questdb/py-questdb-query
[8] https://marp.app
[9] https://blog.streamlit.io/crafting-a-dashboard-app-in-python-using-streamlit/
[10] https://grafana.com/docs/grafana/latest/datasources/opentsdb/
[11] https://docs.influxdata.com/influxdb/cloud/api-guide/client-libraries/python/
[12] https://www.bookstack.cn/read/timescaledb-2.9-en/50d3ef6ad7f1fd7b.md
[13] https://www.reddit.com/r/sre/comments/159zq02/list_of_57_open_source_time_series_databases_you/
[14] https://github.com/dh4/pywu
[15] https://realpython.com/python-dash/
[16] https://www.timescale.com/learn/the-best-time-series-databases-compared
[17] https://github.com/MarcoBuster/WUndergroundPWS-API
[18] https://questdb.com
[19] https://www.iunera.com/kraken/fabric/time-series-database/
[20] https://victoriametrics.com/blog/the-rise-of-open-source-time-series-databases/
[21] https://docs.influxdata.com/influxdb/v2/api-guide/client-libraries/python/
[22] https://docs.influxdata.com/influxdb3/cloud-serverless/query-data/execute-queries/client-libraries/python/
[23] https://siddharthqs.com/introduction-to-timescaledb-for-algorithmic-trading
[24] https://mccarthysean.dev/001-06-adding-data-to-timescale
[25] https://projects.raspberrypi.org/en/projects/uploading-weather-data-to-weather-underground/5
[26] https://zperzan.github.io/projects/scrape-weather-underground/
[27] https://docs.victoriametrics.com
[28] https://leancrew.com/all-this/2014/02/weather-underground-in-pythonista/
[29] https://github.com/idatsy/py-questdb
[30] https://plotly.com/python/time-series/
[31] https://www.tinybird.co/blog-posts/python-real-time-dashboard
[32] https://www.anaconda.com/blog/real-time-dashboard-streaming-refreshing
[33] https://github.com/rafabelokurows/time_series_explorer
[34] https://stackoverflow.com/questions/64828801/plotly-how-to-plot-time-series-in-dash-plotly
[35] https://programminghistorian.org/en/lessons/interactive-data-visualization-dashboard
[36] https://www.reddit.com/r/Python/comments/t9vg96/interactive_timeseries_forecasting_predicting_the/
[37] https://github.com/influxdata/influxdb-client-python
[38] https://influxdb-client.readthedocs.io
[39] https://www.influxdata.com/blog/influxdb-python-client-library-deep-dive-writeapi/
[40] https://pypi.org/project/influxdb/
[41] https://influxdb-client.readthedocs.io/en/v1.2.0/
[42] https://influxdb-python.readthedocs.io/en/latest/examples.html
[43] https://stackoverflow.com/questions/79254344/simple-example-of-influxdb-client-python
[44] https://github.com/timescale/examples
[45] https://www.timescale.com/learn/time-series-analysis-and-forecasting-with-python
[46] https://www.timescale.com/blog/how-to-work-with-time-series-in-python
[47] https://www.youtube.com/watch?v=-I7jprTueFw
[48] https://www.youtube.com/watch?v=JLcU9uAItHk
[49] https://github.com/timescale/examples/blob/master/clients/tsdb-python-client.py
[50] https://stackoverflow.com/questions/60757983/insert-2-million-rows-into-timscaldb-with-python-dataframes

I'll revise the dashboard implementation to use NiceGUI instead of Streamlit, incorporating the latest documentation and best practices. Here's the updated implementation:

Key improvements and NiceGUI-specific features:

1. **Responsive Layout**:
- Uses `ui.row()` and `ui.grid()` for flexible layouts
- Card-based design with consistent padding and spacing
- Full-width elements that adapt to screen size

2. **Real-time Updates**:
- Built-in refresh functionality with `@ui.refreshable`
- Manual refresh button and auto-refresh timer
- Live data updates without page reload

3. **Enhanced Visualization**:
- Interactive Plotly charts with `ui.plotly()`
- AG Grid table with pagination and sorting
- Custom styling with Tailwind classes

4. **Navigation**:
- Fixed left drawer for time range selection
- Persistent header and footer
- Dark mode support

5. **User Experience**:
- Visual feedback with notifications
- Smooth animations and transitions
- Download button with dynamic filename
- Status indicators and loading states

6. **Styling**:
- Custom color schemes
- Consistent spacing and typography
- Card shadows and hover effects
- Responsive breakpoints

To run this dashboard:
```bash
python3 weather_dashboard.py
```

The NiceGUI implementation provides several advantages over the original Streamlit version:
- Persistent state between interactions
- True real-time updates without page reloads
- More flexible layout options
- Better mobile responsiveness
- Customizable theme and styling
- Direct access to browser APIs through Quasar components

For the presentation slides, we can create a Marp deck that highlights these NiceGUI features while maintaining the existing content structure about time series databases and data collection pipelines.

Citations:
[1] https://nicegui.io/documentation
[2] https://www.reddit.com/r/Python/comments/10d6ugv/nicegui_let_any_browser_be_the_frontend_for_your/
[3] https://www.reddit.com/r/nicegui/comments/1dlx3or/fastapi_nicegui_integration/
[4] https://github.com/s71m/nicegui_dashboard
[5] https://nicegui.io/documentation/page
[6] https://github.com/zauberzeug/nicegui/issues/3583
[7] https://nicegui.io/documentation/page_layout
[8] https://nicegui.io/documentation/select
[9] https://nicegui.io/documentation/download
[10] https://www.reddit.com/r/nicegui/comments/1e71di2/documentation_issues/
[11] https://nicegui.io
[12] https://nicegui.io/documentation/section_page_layout
[13] https://nicegui.io/documentation/table
[14] https://www.reddit.com/r/nicegui/comments/13c23l8/nicegui_vs_plotlydash/
[15] https://github.com/zauberzeug/nicegui
[16] https://nicegui.io/documentation/plotly
[17] https://www.datacamp.com/tutorial/nicegui
[18] https://www.bitdoze.com/nicegui-get-started/
[19] https://www.youtube.com/watch?v=RfQ522ReHTo
[20] https://github.com/zauberzeug/nicegui/discussions/2223
[21] https://www.reddit.com/r/nicegui/comments/15pn989/adding_a_title_to_uiaggrid/
[22] https://github.com/zauberzeug/nicegui/discussions/989
[23] https://github.com/zauberzeug/nicegui/discussions/3258
[24] https://docs.webix.com/api__ui.align_left_config.html
[25] https://github.com/zauberzeug/nicegui/discussions/816
[26] https://github.com/zauberzeug/nicegui/discussions/2958
[27] https://nicegui.io/documentation/user
[28] https://stackoverflow.com/questions/79628343/plotly-yaxis-shift-does-not-work-properly
[29] https://github.com/zauberzeug/nicegui/discussions/1358
[30] https://github.com/zauberzeug/nicegui/discussions/765
[31] https://github.com/zauberzeug/nicegui/discussions/2846
[32] https://github.com/zauberzeug/nicegui/discussions/748
[33] https://nicegui.io/documentation/section_pages_routing
[34] https://github.com/zauberzeug/nicegui/discussions/4472
[35] https://www.reddit.com/r/nicegui/comments/192cdxh/how_to_pass_a_dictionary_to_a_uipage/
[36] https://www.reddit.com/r/nicegui/comments/1dceism/how_to_make_the_header_always_visible_in_uitable/
[37] https://github.com/zauberzeug/nicegui/blob/main/examples/single_page_app/main.py
[38] https://github.com/zauberzeug/nicegui/discussions/1883
[39] https://www.reddit.com/r/nicegui/comments/1jbcaak/missing_left_drawer_minimode_transition_animation/
[40] https://www.youtube.com/watch?v=wqyg8bBkX9U
[41] https://stackoverflow.com/questions/44351009/material-ui-left-drawer-in-app-bar-wont-close-on-overlay-click-or-menu-item-cli
[42] https://www.reddit.com/r/nicegui/comments/1e7ayfa/drag_drawer_smallerbigger/
[43] https://www.reddit.com/r/nicegui/comments/1cnm5g6/updating_options_of_uiselect_from_lines_above/
[44] https://nicegui.io/documentation/button_dropdown
[45] https://github.com/zauberzeug/nicegui/discussions/1405
[46] https://github.com/zauberzeug/nicegui/discussions/3049
[47] https://stackoverflow.com/questions/76779219/im-trying-to-make-a-spinner-visible-on-clicking-a-button-but-it-stays-invisible
[48] https://stackoverflow.com/questions/77660820/tooltip-in-ui-table-nicegui
[49] https://stackoverflow.com/questions/76416852/binding-and-updating-values-from-select-option-to-table-row-cells-in-nicegui
[50] https://www.reddit.com/r/nicegui/comments/14wr1io/download_a_data_frame_as_excel/
[51] https://github.com/zauberzeug/nicegui/discussions/3068
[52] https://github.com/zauberzeug/nicegui/discussions/3002
[53] https://www.youtube.com/watch?v=jf-SyFRJLYA
[54] https://www.youtube.com/watch?v=B-45Ps3ZGzc
