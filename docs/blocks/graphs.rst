.. _dom_graphs:

Graphs
======

If your Domoticz device contains a value (temperature, humidity, power, etc.)
then when you click on the block a popup window will appear showing a graph of the values of the device.

Besides popup graphs it's also possible to show the graph directly on the dashboard itself,
by adding the graph-id to a column definition as follows::

    //Adding a graph of device 691 to column 2
    columns[2]['blocks'] = [
      ...,
      'graph_691',      //691 is the device id for which you want to show the graph
      ...
    ]

The following block parameters can be used to configure the graph:

.. list-table:: 
  :header-rows: 1
  :widths: 5 30
  :class: tight-table

  * - Parameter
    - Description
  * - graph
    - | Sets the graph type
      | ``'line'`` Line graph (default)
      | ``'bar'`` Bar graph
  * - graphTypes
    - | Array of values you want to show in the graph. Can be used for Domoticz devices having several values.
      | ``['te']``: Temperature
      | ``['hu']``: Humidity
      | ``['ba']``: Barometer
      | ``['gu', 'sp']``: wind guts and speed
      | ``['uvi']``, ``['lux']``, ``['lux_avg']``, ``['mm']``, ``['v_max']``
      | ``['v2']``, ``['mm']``, ``['eu']``, ``['u']``, ``['u_max']``,``['co2']``
  * - height
    - ``'300px'``: Height of the graph in the graph block
  * - displayFormats
    - Object to set the time display format on the x-axis. See below for an example.
  * - custom
    - Customized graph. See below for examples

We will show the possibilities by showing a:

* Simple energy device (Solar panel)
* Climate device (temperature, humidity, barometer)
* P1 Smart Meter

Simple energy device
--------------------

The solar panel device has device id 6. First we add it to a column without any additional configuration parameters,
to show the default result::

  columns[2]['blocks'] = [
    6
  ]


.. image :: img/solar_default.jpg

As you see three buttons are generated (actual power, energy today, total energy).
I only want to have one button, so I change my column definition to::

  columns[2]['blocks'] = [
    '6_1'
  ]

By pressing the button the following graphs pops up:

.. image :: img/solar_1_default.jpg

So, nothing special. Only the red line color is maybe a bit too much. Let's change it into a yellow bar graph.
We have to add a block definition::

    blocks['graph_6'] = {
        graph: 'bar',
        datasetColors: ['yellow']
    }

.. image :: img/solar_yellow_bar.jpg

Now I want to add a legend at the bottom::

    blocks['graph_6'] = {
        graph: 'bar',
        datasetColors: ['yellow'],
        legend: true
    }

.. image :: img/solar_legend.jpg

As you can see the data points are labeled as 'V'. This name is generated by Domoticz. We can translate the Domoticz name in something else, by extending the legend parameter ::

    blocks['graph_6'] = {
        graph: 'bar',
        datasetColors: ['yellow'],
        legend: {
          'v': 'Power generation'
        }
    }

``legend`` is an object consisting of key-value pairs for the translation from Domoticz names to custom names.

After pressing the 'Month' button on the popup graph:

.. image :: img/solar_custom_legend.jpg

Climate device
--------------
First let's add a climate device with Domoticz ID 659 to a column::

    columns[3]['blocks'] = [
        'graph_659'
    ]

This will show the graph directly on the Dashticz dashboard:

.. image :: img/climate.jpg

As you can see the climate device has three subdevices (temperature, humidity, pressure).
Since these are different properties three Y axes are being created.

If you prefer to only see the temperature and humidity add a block definition::

    blocks['graph_659'] = {
        graphTypes : ['te', 'hu'],
        legend: true
    }


.. image :: img/climate_te_hu.jpg

Of course you can add a legend as well. See the previous section for an example.

P1 smart meter
--------------

First let's show the default P1 smart meter graph:

.. image :: img/p1.jpg

You see a lot of lines. What do they mean? Let's add a legend ::

    blocks['graph_43'] = {
        legend: true
    }

This gives:

.. image :: img/p1_legend.jpg

That doesn't tell too much. However, this are the value names as provided by Domoticz.
Now you have to know that a P1 power meter has 4 values:

* Power usage tariff 1
* Power usage tariff 2
* Power delivery tariff 1
* Power delivery tariff 2

The first two represent the energy that flows into your house. The last two represent the energy that your house delivers back to the grid.

So we can add a more meaningful legend as follows::

    blocks['graph_43'] = {
        legend: {
          v: "Usage 1",
          v2: "Usage 2",
          r1: "Return 1",
          r2: "Return 2"
    }

Resulting in:

.. image :: img/p1_legend_2.jpg

However what I would like to see is:

* The sum of Usage 1 and Usage 2
* The sum of Return 1 and Return 2, but then negative
* A line to show the nett energy usage: Usage 1 + Usage 2 - Return 1 - Return
* The usage and return data should be presented as bars. The nett energy as a line.

Can we do that? Yes, with custom graphs!

Custom graphs
-------------

I use the P1 smart meter as an example again to demonstrate how to create custom graphs. First the code and result.
The explanation will follow after that::

    blocks['graph_43'] = {
        title: 'My Power',
        graph: ['line','bar','bar'], 
        custom : {
            "last day": {
                range: 'day',
                filter: '24 hours',
                data: {
                    nett: 'd.v+d.v2-d.r1-d.r2',
                    usage: 'd.v+d.v2',
                    generation: '-d.r1-d.r2'
                }
            },
            "last 2 weeks": {
                range: 'month',
                filter: '14 days',
                data: {
                    nett: 'd.v+d.v2-d.r1-d.r2',
                    usage: 'd.v+d.v2',
                    generation: '-d.r1-d.r2'
                }
            },
            "last 6 months": {
                range: 'year',
                filter: '6 months',
                data: {
                    nett: 'd.v+d.v2-d.r1-d.r2',
                    usage: 'd.v+d.v2',
                    generation: '-d.r1-d.r2'
                }
            }
        },
        legend: true,
        datasetColors:['blue','red','yellow']
    }

This will give:

.. image :: img/p1_custom.jpg

As you can see, the graph has

* title, set via the ``title`` parameter
* custom colors, defined by the parameter ``datasetColors``
* The ``graph`` parameter is used to define the graph types. This time it's an array, because we want to select the graph type per value.
* ``legend`` set to true, to show a default legend
* custom buttons, defined by the ``custom`` parameter

A ``custom`` object start with the name of the button. The button should contain the following three parameters:

* ``range``. This is the name of the range as requested from Domoticz, and can be ``'day'``, ``'month'`` or ``'year'``.
* ``filter`` (optional). This limits the amount of data to the period as defined by this parameter. Examples: ``'2 hours'``, ``'4 days'``, ``'3 months'``
* ``data``. This is an object that defines the values to use for the graph.

As you can see in the example the first value will have the name 'nett'. The formula to compute the value is::

  'd.v+d.v2-d.r1-d.r2'

The ``d`` object contains the data as delivered by Domoticz. As you maybe remember from a previous example
Domoticz provides the two power usage values (v and v2) and the two power return values (r1 and r2).

So the first part sums the two power usage values (``d.v+d.v2``) and the last parts substracts the two return values (``-d.r1-d.r2``),

The two other value-names in the data object (usage and generation) will compute the sum of the power usage values and the power return values respectively.

Maybe a bit complex in the beginning, but the Dashticz forum is not far away.

Below another example to adapt the reported values of a watermeter to liters::

    blocks['graph_903'] = {
        graph: 'bar',
        datasetColors: ['lightblue'],
        legend: true,
        custom : {
            "last hours": {
                range: 'day',
                filter: '6 hours',
                data: {
                    liter: 'd.v*100'            }
                },

      "today": {
                range: 'day',
                filter: '12 hours',
                data: {
                    liter: 'd.v*100'            }
                },
      
      "last week": {
                range: 'month',
                filter: '7 days',
                data: {
                    liter: 'd.v*1000'            }
                }


            }
      }

.. image :: img/water.jpg


Time format on the x-axis
-------------------------

The chart module uses moments.js for displaying the times and dates.
The locale will be set via the Domoticz setting for the calendar language::

  config['calendarlanguage'] = 'nl_NL';

To set the time (or date) format for the x-axis add the ``displayFormats`` parameter to the block definition::

    blocks['graph_6'] = {
        displayFormats : {
          minute: 'h:mm a',
          hour: 'hA',
          day: 'MMM D',
          week: 'll',
          month: 'MMM D',
        },
    }

The previous example sets the time formats to UK style. See https://www.chartjs.org/docs/latest/axes/cartesian/time.html#display-formats for time/date formats. 

Modifying the y-axes
--------------------

You can modify the y-axes by setting the options parameter. Below you see an example how to define the min and max values of two y-axes::

    blocks['graph_659'] = {
        graph: 'line',
        graphTypes: ['te', 'hu'],
        options: {
            scales: {
                yAxes: [{
                    ticks: {
                        min: 0,
                        max: 30
                    }
                }, {
                    ticks: {
                        min: 50,
                        max: 100
                    }
                }]
            }
        }
    }

The ``yAxes`` parameter in the ``options`` block is an array, with an entry for each y-axis.

Y-axis for custom graphs
------------------------

To define the y-axes for a custom graph you can add the ``ylabels`` parameter as follows::

    blocks['graph_659'] = {
        custom: {
            'The Temp': {
                ylabels: ['yaxis of temp'],
                data: {
                    'temp value': 'd.te'
                },
                range: 'day',
                filter: '2 days',
                legend: true
            }
        },
        width: 6
    }

.. image :: img/customlabels.jpg

The parameter ``ylabels`` is an array. You can add a string for each value of the data object. 

Styling
-------

For graphs the following css-classes are used:

* .graph_header: The graph header, including title and buttons
* .graph_title: The title of the graph, including the current value
* .graph_buttons: The buttons for the graph

You can modify the class definition in custom.css. If you want to hide the header::

  .graph_header {
    display: none;
  }

You can also modify the class for a specific graph only ::

  .block_graph_43 .graph_header {
    display: none;
  }

In the previous example only the graph for device id 43 will be affected.

To change the default size of the graph popup windows add the following style blocks to your custom.css::

    .graphheight {
      height: 400px;
    }
    
    .graphwidth {
      width: 400px;
    }

To remove the close button of the graph popup add the following text to custom.css::

    .graphclose { display: none; }



To be detailed... ::

    .opengraph, .opengraph<idx>p, #opengraph<idx>p   //classes attached to the graph popup dialog
    .graphcurrent<idx>      //class attached to the div with the current value

For internal use::

    block_graph_<idx>     //The div to which the graph needs to be attached.
    #graphoutput<idx>     //The canvas for the graph output

  
