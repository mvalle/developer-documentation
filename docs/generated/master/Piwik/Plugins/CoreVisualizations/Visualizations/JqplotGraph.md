<small>Piwik\Plugins\CoreVisualizations\Visualizations\</small>

JqplotGraph
===========

DataTable visualization that displays DataTable data in a JQPlot graph.

TODO: should merge all this logic w/ jqplotdatagenerator & 'Chart' visualizations.

Properties
----------

This abstract class defines the following properties:

- [`$config`](#$config) &mdash; JqplotGraph\Config$config
- [`$requestConfig`](#$requestconfig) &mdash; Contains request properties for this visualization. Inherited from [`ViewDataTable`](../../../../Piwik/Plugin/ViewDataTable.md)

<a name="$config" id="$config"></a>
<a name="config" id="config"></a>
### `$config`

JqplotGraph\Config$config

#### Signature

- It is a `JqplotGraph\Config` value.

<a name="$requestconfig" id="$requestconfig"></a>
<a name="requestConfig" id="requestConfig"></a>
### `$requestConfig`

Contains request properties for this visualization.

#### Signature

- It is a [`RequestConfig`](../../../../Piwik/ViewDataTable/RequestConfig.md) value.
