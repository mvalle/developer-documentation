Events
==========

This is a complete list of available hooks.

## Access

- [Access.createAccessSingleton](#accesscreateaccesssingleton)

### Access.createAccessSingleton
_Defined in [Piwik/Access](https://github.com/piwik/piwik/blob/master/core/Access.php) in line [46](https://github.com/piwik/piwik/blob/master/core/Access.php#L46)_



Callback Signature:
<pre><code>function(&amp;self::$instance)</code></pre>

## API

- [API.$pluginName.$methodName](#apipluginnamemethodname)
- [API.$pluginName.$methodName.end](#apipluginnamemethodnameend)
- [API.getReportMetadata.end](#apigetreportmetadataend)
- [API.getSegmentDimensionMetadata](#apigetsegmentdimensionmetadata)
- [API.Request.authenticate](#apirequestauthenticate)
- [API.Request.dispatch](#apirequestdispatch)
- [API.Request.dispatch.end](#apirequestdispatchend)

### API.$pluginName.$methodName
_Defined in [Piwik/API/Proxy](https://github.com/piwik/piwik/blob/master/core/API/Proxy.php) in line [206](https://github.com/piwik/piwik/blob/master/core/API/Proxy.php#L206)_

Triggered before an API request is dispatched. This event exists for convenience and is triggered directly after the [API.Request.dispatch](/api-reference/events#apirequestdispatch)
event is triggered. It can be used to modify the arguments passed to a **single** API method.

_Note: This is can be accomplished with the [API.Request.dispatch](/api-reference/events#apirequestdispatch) event as well, however
event handlers for that event will have to do more work._

**Example**

    Piwik::addAction('API.Actions.getPageUrls', function (&$parameters) {
        // force use of a single website. for some reason.
        $parameters['idSite'] = 1;
    });

Callback Signature:
<pre><code>function(&amp;$finalParameters)</code></pre>

- `array` `&$finalParameters` List of parameters that will be passed to the API method.


### API.$pluginName.$methodName.end
_Defined in [Piwik/API/Proxy](https://github.com/piwik/piwik/blob/master/core/API/Proxy.php) in line [256](https://github.com/piwik/piwik/blob/master/core/API/Proxy.php#L256)_

Triggered directly after an API request is dispatched. This event exists for convenience and is triggered immediately before the
[API.Request.dispatch.end](/api-reference/events#apirequestdispatchend) event. It can be used to modify the output of a **single**
API method.

_Note: This can be accomplished with the [API.Request.dispatch.end](/api-reference/events#apirequestdispatchend) event as well,
however event handlers for that event will have to do more work._

**Example**

    // append (0 hits) to the end of row labels whose row has 0 hits
    Piwik::addAction('API.Actions.getPageUrls', function (&$returnValue, $info)) {
        $returnValue->filter('ColumnCallbackReplace', 'label', function ($label, $hits) {
            if ($hits === 0) {
                return $label . " (0 hits)";
            } else {
                return $label;
            }
        }, null, array('nb_hits'));
    }

Callback Signature:
<pre><code>$endHookParams</code></pre>

- `mixed` `$returnedValue` The API method's return value. Can be an object, such as a [DataTable](/api-reference/Piwik/DataTable) instance. could be a [DataTable](/api-reference/Piwik/DataTable).

- `array` `$extraInfo` An array holding information regarding the API request. Will contain the following data: - **className**: The namespace-d class name of the API instance that's being called. - **module**: The name of the plugin the API request was dispatched to. - **action**: The name of the API method that was executed. - **parameters**: The array of parameters passed to the API method.


### API.getReportMetadata.end
_Defined in [Piwik/Plugins/API/ProcessedReport](https://github.com/piwik/piwik/blob/master/plugins/API/ProcessedReport.php) in line [259](https://github.com/piwik/piwik/blob/master/plugins/API/ProcessedReport.php#L259)_

Triggered after all available reports are collected. This event can be used to modify the report metadata of reports in other plugins. You
could, for example, add custom metrics to every report or remove reports from the list
of available reports.

Callback Signature:
<pre><code>function(&amp;$availableReports, $parameters)</code></pre>

- `array` `&$availableReports` List of all report metadata. Read the [API.getReportMetadata](/api-reference/events#apigetreportmetadata) docs to see what this array contains.

- `array` `$parameters` Contains the values of the sites and period we are getting reports for. Some report depend on this data. For example, Goals reports depend on the site IDs being request. Contains the following information: - **idSites**: The array of site IDs we are getting reports for. - **period**: The period type, eg, `'day'`, `'week'`, `'month'`, `'year'`, `'range'`. - **date**: A string date within the period or a date range, eg, `'2013-01-01'` or `'2012-01-01,2013-01-01'`.

Usages:

[Goals::getReportMetadataEnd](https://github.com/piwik/piwik/blob/master/plugins/Goals/Goals.php#L148)


### API.getSegmentDimensionMetadata
_Defined in [Piwik/Plugins/API/API](https://github.com/piwik/piwik/blob/master/plugins/API/API.php) in line [150](https://github.com/piwik/piwik/blob/master/plugins/API/API.php#L150)_

Triggered when gathering all available segment dimensions. This event can be used to make new segment dimensions available.

**Example**

    public function getSegmentsMetadata(&$segments, $idSites)
    {
        $segments[] = array(
            'type'           => 'dimension',
            'category'       => Piwik::translate('General_Visit'),
            'name'           => 'General_VisitorIP',
            'segment'        => 'visitIp',
            'acceptedValues' => '13.54.122.1, etc.',
            'sqlSegment'     => 'log_visit.location_ip',
            'sqlFilter'      => array('Piwik\IP', 'P2N'),
            'permission'     => $isAuthenticatedWithViewAccess,
        );
    }

Callback Signature:
<pre><code>function(&amp;$segments, $idSites)</code></pre>

- `array` `$dimensions` The list of available segment dimensions. Append to this list to add new segments. Each element in this list must contain the following information: - **type**: Either `'metric'` or `'dimension'`. `'metric'` means the value is a numeric and `'dimension'` means it is a string. Also, `'metric'` values will be displayed under **Visit (metrics)** in the Segment Editor. - **category**: The segment category name. This can be an existing segment category visible in the segment editor. - **name**: The pretty name of the segment. Can be a translation token. - **segment**: The segment name, eg, `'visitIp'` or `'searches'`. - **acceptedValues**: A string describing one or two exacmple values, eg `'13.54.122.1, etc.'`. - **sqlSegment**: The table column this segment will segment by. For example, `'log_visit.location_ip'` for the **visitIp** segment. - **sqlFilter**: A PHP callback to apply to segment values before they are used in SQL. - **permission**: True if the current user has view access to this segment, false if otherwise.

- `array` `$idSites` The list of site IDs we're getting the available segments for. Some segments (such as Goal segments) depend on the site.

Usages:

[CustomVariables::getSegmentsMetadata](https://github.com/piwik/piwik/blob/master/plugins/CustomVariables/CustomVariables.php#L79)


### API.Request.authenticate
_Defined in [Piwik/API/Request](https://github.com/piwik/piwik/blob/master/core/API/Request.php) in line [260](https://github.com/piwik/piwik/blob/master/core/API/Request.php#L260)_

Triggered when authenticating an API request, but only if the **token_auth** query parameter is found in the request. Plugins that provide authentication capabilities should subscribe to this event
and make sure the global authentication object (the object returned by `Registry::get('auth')`)
is setup to use `$token_auth` when its `authenticate()` method is executed.

Callback Signature:
<pre><code>function($token_auth)</code></pre>

- `string` `$token_auth` The value of the **token_auth** query parameter.

Usages:

[Login::ApiRequestAuthenticate](https://github.com/piwik/piwik/blob/master/plugins/Login/Login.php#L59)


### API.Request.dispatch
_Defined in [Piwik/API/Proxy](https://github.com/piwik/piwik/blob/master/core/API/Proxy.php) in line [186](https://github.com/piwik/piwik/blob/master/core/API/Proxy.php#L186)_

Triggered before an API request is dispatched. This event can be used to modify the arguments passed to one or more API methods.

**Example**

    Piwik::addAction('API.Request.dispatch', function (&$parameters, $pluginName, $methodName) {
        if ($pluginName == 'Actions') {
            if ($methodName == 'getPageUrls') {
                // ... do something ...
            } else {
                // ... do something else ...
            }
        }
    });

Callback Signature:
<pre><code>function(&amp;$finalParameters, $pluginName, $methodName)</code></pre>

- `array` `&$finalParameters` List of parameters that will be passed to the API method.

- `string` `$pluginName` The name of the plugin the API method belongs to.

- `string` `$methodName` The name of the API method that will be called.


### API.Request.dispatch.end
_Defined in [Piwik/API/Proxy](https://github.com/piwik/piwik/blob/master/core/API/Proxy.php) in line [296](https://github.com/piwik/piwik/blob/master/core/API/Proxy.php#L296)_

Triggered directly after an API request is dispatched. This event can be used to modify the output of any API method.

**Example**

    // append (0 hits) to the end of row labels whose row has 0 hits for any report that has the 'nb_hits' metric
    Piwik::addAction('API.Actions.getPageUrls', function (&$returnValue, $info)) {
        // don't process non-DataTable reports and reports that don't have the nb_hits column
        if (!($returnValue instanceof DataTableInterface)
            || in_array('nb_hits', $returnValue->getColumns())
        ) {
            return;
        }

        $returnValue->filter('ColumnCallbackReplace', 'label', function ($label, $hits) {
            if ($hits === 0) {
                return $label . " (0 hits)";
            } else {
                return $label;
            }
        }, null, array('nb_hits'));
    }

Callback Signature:
<pre><code>$endHookParams</code></pre>

- `mixed` `$returnedValue` The API method's return value. Can be an object, such as a [DataTable](/api-reference/Piwik/DataTable) instance.

- `array` `$extraInfo` An array holding information regarding the API request. Will contain the following data: - **className**: The namespace-d class name of the API instance that's being called. - **module**: The name of the plugin the API request was dispatched to. - **action**: The name of the API method that was executed. - **parameters**: The array of parameters passed to the API method.

## ArchiveProcessor

- [ArchiveProcessor.Parameters.getIdSites](#archiveprocessorparametersgetidsites)

### ArchiveProcessor.Parameters.getIdSites
_Defined in [Piwik/ArchiveProcessor/Parameters](https://github.com/piwik/piwik/blob/master/core/ArchiveProcessor/Parameters.php) in line [110](https://github.com/piwik/piwik/blob/master/core/ArchiveProcessor/Parameters.php#L110)_



Callback Signature:
<pre><code>function(&amp;$idSites, $this-&gt;getPeriod())</code></pre>

## AssetManager

- [AssetManager.filterMergedJavaScripts](#assetmanagerfiltermergedjavascripts)
- [AssetManager.filterMergedStylesheets](#assetmanagerfiltermergedstylesheets)
- [AssetManager.getJavaScriptFiles](#assetmanagergetjavascriptfiles)
- [AssetManager.getStylesheetFiles](#assetmanagergetstylesheetfiles)

### AssetManager.filterMergedJavaScripts
_Defined in [Piwik/AssetManager/UIAssetMerger/JScriptUIAssetMerger](https://github.com/piwik/piwik/blob/master/core/AssetManager/UIAssetMerger/JScriptUIAssetMerger.php) in line [71](https://github.com/piwik/piwik/blob/master/core/AssetManager/UIAssetMerger/JScriptUIAssetMerger.php#L71)_

Triggered after all the JavaScript files Piwik uses are minified and merged into a single file, but before the merged JavaScript is written to disk. Plugins can use this event to modify merged JavaScript or do something else
with it.

Callback Signature:
<pre><code>function(&amp;$mergedContent)</code></pre>

- `string` `&$mergedContent` The minified and merged JavaScript.


### AssetManager.filterMergedStylesheets
_Defined in [Piwik/AssetManager/UIAssetMerger/StylesheetUIAssetMerger](https://github.com/piwik/piwik/blob/master/core/AssetManager/UIAssetMerger/StylesheetUIAssetMerger.php) in line [74](https://github.com/piwik/piwik/blob/master/core/AssetManager/UIAssetMerger/StylesheetUIAssetMerger.php#L74)_

Triggered after all less stylesheets are compiled to CSS, minified and merged into one file, but before the generated CSS is written to disk. This event can be used to modify merged CSS.

Callback Signature:
<pre><code>function(&amp;$mergedContent)</code></pre>

- `string` `&$mergedContent` The merged and minified CSS.


### AssetManager.getJavaScriptFiles
_Defined in [Piwik/AssetManager/UIAssetFetcher/JScriptUIAssetFetcher](https://github.com/piwik/piwik/blob/master/core/AssetManager/UIAssetFetcher/JScriptUIAssetFetcher.php) in line [47](https://github.com/piwik/piwik/blob/master/core/AssetManager/UIAssetFetcher/JScriptUIAssetFetcher.php#L47)_

Triggered when gathering the list of all JavaScript files needed by Piwik and its plugins. Plugins that have their own JavaScript should use this event to make those
files load in the browser.

JavaScript files should be placed within a **javascripts** subdirectory in your
plugin's root directory.

_Note: While you are developing your plugin you should enable the config setting
`[Development] disable_merged_assets` so JavaScript files will be reloaded immediately
after every change._

**Example**

    public function getJsFiles(&$jsFiles)
    {
        $jsFiles[] = "plugins/MyPlugin/javascripts/myfile.js";
        $jsFiles[] = "plugins/MyPlugin/javascripts/anotherone.js";
    }

Callback Signature:
<pre><code>function(&amp;$this-&gt;fileLocations)</code></pre>

- `string` `$jsFiles` The JavaScript files to load.

Usages:

[Actions::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/Actions/Actions.php#L98), [Annotations::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/Annotations/Annotations.php#L40), [CoreAdminHome::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/CoreAdminHome/CoreAdminHome.php#L46), [CoreHome::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/CoreHome/CoreHome.php#L51), [CorePluginsAdmin::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/CorePluginsAdmin/CorePluginsAdmin.php#L44), [CoreVisualizations::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/CoreVisualizations/CoreVisualizations.php#L58), [Dashboard::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/Dashboard/Dashboard.php#L205), [ExamplePlugin::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/ExamplePlugin/ExamplePlugin.php#L25), [Feedback::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/Feedback/Feedback.php#L36), [Goals::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/Goals/Goals.php#L254), [Insights::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/Insights/Insights.php#L31), [LanguagesManager::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/LanguagesManager/LanguagesManager.php#L47), [Live::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/Live/Live.php#L39), [Login::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/Login/Login.php#L39), [MobileMessaging::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/MobileMessaging/MobileMessaging.php#L86), [MultiSites::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/MultiSites/MultiSites.php#L73), [Overlay::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/Overlay/Overlay.php#L37), [PrivacyManager::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/PrivacyManager/PrivacyManager.php#L149), [ScheduledReports::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/ScheduledReports.php#L115), [SegmentEditor::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/SegmentEditor/SegmentEditor.php#L92), [SitesManager::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/SitesManager/SitesManager.php#L45), [Transitions::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/Transitions/Transitions.php#L33), [UserCountry::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/UserCountry/UserCountry.php#L57), [UserCountryMap::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/UserCountryMap/UserCountryMap.php#L59), [UsersManager::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/UsersManager/UsersManager.php#L81), [Widgetize::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/Widgetize/Widgetize.php#L27), [ZenMode::getJsFiles](https://github.com/piwik/piwik/blob/master/plugins/ZenMode/ZenMode.php#L39)


### AssetManager.getStylesheetFiles
_Defined in [Piwik/AssetManager/UIAssetFetcher/StylesheetUIAssetFetcher](https://github.com/piwik/piwik/blob/master/core/AssetManager/UIAssetFetcher/StylesheetUIAssetFetcher.php) in line [67](https://github.com/piwik/piwik/blob/master/core/AssetManager/UIAssetFetcher/StylesheetUIAssetFetcher.php#L67)_

Triggered when gathering the list of all stylesheets (CSS and LESS) needed by Piwik and its plugins. Plugins that have stylesheets should use this event to make those stylesheets
load.

Stylesheets should be placed within a **stylesheets** subdirectory in your plugin's
root directory.

**Example**

    public function getStylesheetFiles(&$stylesheets)
    {
        $stylesheets[] = "plugins/MyPlugin/stylesheets/myfile.less";
        $stylesheets[] = "plugins/MyPlugin/stylesheets/myotherfile.css";
    }

Callback Signature:
<pre><code>function(&amp;$this-&gt;fileLocations)</code></pre>

- `string` `$stylesheets` The list of stylesheet paths.

Usages:

[Plugin::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/API/API.php#L612), [Actions::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/Actions/Actions.php#L93), [Annotations::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/Annotations/Annotations.php#L32), [CoreAdminHome::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/CoreAdminHome/CoreAdminHome.php#L36), [CoreHome::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/CoreHome/CoreHome.php#L28), [CorePluginsAdmin::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/CorePluginsAdmin/CorePluginsAdmin.php#L28), [CoreVisualizations::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/CoreVisualizations/CoreVisualizations.php#L52), [DBStats::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/DBStats/DBStats.php#L30), [Dashboard::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/Dashboard/Dashboard.php#L214), [ExampleRssWidget::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/ExampleRssWidget/ExampleRssWidget.php#L26), [Feedback::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/Feedback/Feedback.php#L29), [Goals::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/Goals/Goals.php#L259), [Insights::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/Insights/Insights.php#L26), [Installation::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/Installation/Installation.php#L106), [LanguagesManager::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/LanguagesManager/LanguagesManager.php#L42), [Live::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/Live/Live.php#L33), [MobileMessaging::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/MobileMessaging/MobileMessaging.php#L91), [MultiSites::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/MultiSites/MultiSites.php#L82), [SegmentEditor::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/SegmentEditor/SegmentEditor.php#L97), [SitesManager::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/SitesManager/SitesManager.php#L36), [Transitions::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/Transitions/Transitions.php#L28), [UserCountry::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/UserCountry/UserCountry.php#L52), [UserCountryMap::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/UserCountryMap/UserCountryMap.php#L69), [UsersManager::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/UsersManager/UsersManager.php#L90), [VisitsSummary::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/VisitsSummary/VisitsSummary.php#L30), [Widgetize::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/Widgetize/Widgetize.php#L39), [ZenMode::getStylesheetFiles](https://github.com/piwik/piwik/blob/master/plugins/ZenMode/ZenMode.php#L46)

## Config

- [Config.badConfigurationFile](#configbadconfigurationfile)
- [Config.NoConfigurationFile](#confignoconfigurationfile)

### Config.badConfigurationFile
_Defined in [Piwik/FrontController](https://github.com/piwik/piwik/blob/master/core/FrontController.php) in line [372](https://github.com/piwik/piwik/blob/master/core/FrontController.php#L372)_

Triggered when Piwik cannot access database data. This event can be used to start the installation process or to display a custom error
message.

Callback Signature:
<pre><code>function($exception)</code></pre>

- `[Exception](http://php.net/class.Exception)` `$exception` The exception thrown from trying to get an option value.

Usages:

[Installation::dispatch](https://github.com/piwik/piwik/blob/master/plugins/Installation/Installation.php#L82)


### Config.NoConfigurationFile
_Defined in [Piwik/FrontController](https://github.com/piwik/piwik/blob/master/core/FrontController.php) in line [276](https://github.com/piwik/piwik/blob/master/core/FrontController.php#L276)_

Triggered when the configuration file cannot be found or read, which usually means Piwik is not installed yet. This event can be used to start the installation process or to display a custom error message.

Callback Signature:
<pre><code>function($exception)</code></pre>

- `[Exception](http://php.net/class.Exception)` `$exception` The exception that was thrown by `Config::getInstance()`.

Usages:

[Installation::dispatch](https://github.com/piwik/piwik/blob/master/plugins/Installation/Installation.php#L82)

## Console

- [Console.filterCommands](#consolefiltercommands)

### Console.filterCommands
_Defined in [Piwik/Console](https://github.com/piwik/piwik/blob/master/core/Console.php) in line [94](https://github.com/piwik/piwik/blob/master/core/Console.php#L94)_

Triggered to filter / restrict console commands. Plugins that want to restrict commands
should subscribe to this event and remove commands from the existing list.

**Example**

    public function filterConsoleCommands(&$commands)
    {
        $key = array_search('Piwik\Plugins\MyPlugin\Commands\MyCommand', $commands);
        if (false !== $key) {
            unset($commands[$key]);
        }
    }

Callback Signature:
<pre><code>function(&amp;$commands)</code></pre>

- `array` `&$commands` An array containing a list of command class names.

## Controller

- [Controller.$module.$action](#controllermoduleaction)
- [Controller.$module.$action.end](#controllermoduleactionend)

### Controller.$module.$action
_Defined in [Piwik/FrontController](https://github.com/piwik/piwik/blob/master/core/FrontController.php) in line [574](https://github.com/piwik/piwik/blob/master/core/FrontController.php#L574)_

Triggered directly before controller actions are dispatched. This event exists for convenience and is triggered directly after the [Request.dispatch](/api-reference/events#requestdispatch)
event is triggered.

It can be used to do the same things as the [Request.dispatch](/api-reference/events#requestdispatch) event, but for one controller
action only. Using this event will result in a little less code than [Request.dispatch](/api-reference/events#requestdispatch).

Callback Signature:
<pre><code>function(&amp;$parameters)</code></pre>

- `array` `&$parameters` The arguments passed to the controller action.


### Controller.$module.$action.end
_Defined in [Piwik/FrontController](https://github.com/piwik/piwik/blob/master/core/FrontController.php) in line [591](https://github.com/piwik/piwik/blob/master/core/FrontController.php#L591)_

Triggered after a controller action is successfully called. This event exists for convenience and is triggered immediately before the [Request.dispatch.end](/api-reference/events#requestdispatchend)
event is triggered.

It can be used to do the same things as the [Request.dispatch.end](/api-reference/events#requestdispatchend) event, but for one
controller action only. Using this event will result in a little less code than
[Request.dispatch.end](/api-reference/events#requestdispatchend).

Callback Signature:
<pre><code>function(&amp;$result, $parameters)</code></pre>

- `mixed` `&$result` The result of the controller action.

- `array` `$parameters` The arguments passed to the controller action.

## CronArchive

- [CronArchive.archiveSingleSite.finish](#cronarchivearchivesinglesitefinish)
- [CronArchive.archiveSingleSite.start](#cronarchivearchivesinglesitestart)
- [CronArchive.filterWebsiteIds](#cronarchivefilterwebsiteids)
- [CronArchive.init.finish](#cronarchiveinitfinish)

### CronArchive.archiveSingleSite.finish
_Defined in [Piwik/CronArchive](https://github.com/piwik/piwik/blob/master/core/CronArchive.php) in line [335](https://github.com/piwik/piwik/blob/master/core/CronArchive.php#L335)_

This event is triggered immediately after the cron archiving process starts archiving data for a single site.

Callback Signature:
<pre><code>function($idSite, $completed)</code></pre>

- `int` `$idSite` The ID of the site we're archiving data for.


### CronArchive.archiveSingleSite.start
_Defined in [Piwik/CronArchive](https://github.com/piwik/piwik/blob/master/core/CronArchive.php) in line [325](https://github.com/piwik/piwik/blob/master/core/CronArchive.php#L325)_

This event is triggered before the cron archiving process starts archiving data for a single site.

Callback Signature:
<pre><code>function($idSite)</code></pre>

- `int` `$idSite` The ID of the site we're archiving data for.


### CronArchive.filterWebsiteIds
_Defined in [Piwik/CronArchive](https://github.com/piwik/piwik/blob/master/core/CronArchive.php) in line [929](https://github.com/piwik/piwik/blob/master/core/CronArchive.php#L929)_

Triggered by the **core:archive** console command so plugins can modify the list of websites that the archiving process will be launched for. Plugins can use this hook to add websites to archive, remove websites to archive, or change
the order in which websites will be archived.

Callback Signature:
<pre><code>function(&amp;$websiteIds)</code></pre>

- `array` `&$websiteIds` The list of website IDs to launch the archiving process for.


### CronArchive.init.finish
_Defined in [Piwik/CronArchive](https://github.com/piwik/piwik/blob/master/core/CronArchive.php) in line [270](https://github.com/piwik/piwik/blob/master/core/CronArchive.php#L270)_

This event is triggered after a CronArchive instance is initialized.

Callback Signature:
<pre><code>function($this-&gt;websites-&gt;getInitialSiteIds())</code></pre>

- `array` `$websiteIds` The list of website IDs this CronArchive instance is processing. This will be the entire list of IDs regardless of whether some have already been processed.

## Dashboard

- [Dashboard.changeDefaultDashboardLayout](#dashboardchangedefaultdashboardlayout)

### Dashboard.changeDefaultDashboardLayout
_Defined in [Piwik/Plugins/Dashboard/Dashboard](https://github.com/piwik/piwik/blob/master/plugins/Dashboard/Dashboard.php) in line [101](https://github.com/piwik/piwik/blob/master/plugins/Dashboard/Dashboard.php#L101)_

Allows other plugins to modify the default dashboard layout.

Callback Signature:
<pre><code>function(&amp;$defaultLayout)</code></pre>

- `string` `&$defaultLayout` JSON encoded string of the default dashboard layout. Contains an array of columns where each column is an array of widgets. Each widget is an associative array w/ the following elements: * **uniqueId**: The widget's unique ID. * **parameters**: The array of query parameters that should be used to get this widget's report.

## Db

- [Db.cannotConnectToDb](#dbcannotconnecttodb)
- [Db.getDatabaseConfig](#dbgetdatabaseconfig)

### Db.cannotConnectToDb
_Defined in [Piwik/FrontController](https://github.com/piwik/piwik/blob/master/core/FrontController.php) in line [349](https://github.com/piwik/piwik/blob/master/core/FrontController.php#L349)_

Triggered when Piwik cannot connect to the database. This event can be used to start the installation process or to display a custom error
message.

Callback Signature:
<pre><code>function($exception)</code></pre>

- `[Exception](http://php.net/class.Exception)` `$exception` The exception thrown from creating and testing the database connection.

Usages:

[Installation::displayDbConnectionMessage](https://github.com/piwik/piwik/blob/master/plugins/Installation/Installation.php#L40)


### Db.getDatabaseConfig
_Defined in [Piwik/Db](https://github.com/piwik/piwik/blob/master/core/Db.php) in line [82](https://github.com/piwik/piwik/blob/master/core/Db.php#L82)_

Triggered before a database connection is established. This event can be used to change the settings used to establish a connection.

Callback Signature:
<pre><code>function(&amp;$dbConfig)</code></pre>

- `array`

## Goals

- [Goals.getReportsWithGoalMetrics](#goalsgetreportswithgoalmetrics)

### Goals.getReportsWithGoalMetrics
_Defined in [Piwik/Plugins/Goals/Goals](https://github.com/piwik/piwik/blob/master/plugins/Goals/Goals.php) in line [225](https://github.com/piwik/piwik/blob/master/plugins/Goals/Goals.php#L225)_

Triggered when gathering all reports that contain Goal metrics. The list of reports
will be displayed on the left column of the bottom of every _Goals_ page.

If plugins define reports that contain goal metrics (such as **conversions** or **revenue**),
they can use this event to make sure their reports can be viewed on Goals pages.

**Example**

    public function getReportsWithGoalMetrics(&$reports)
    {
        $reports[] = array(
            'category' => Piwik::translate('MyPlugin_myReportCategory'),
            'name' => Piwik::translate('MyPlugin_myReportDimension'),
            'module' => 'MyPlugin',
            'action' => 'getMyReport'
        );
    }

Callback Signature:
<pre><code>function(&amp;$reportsWithGoals)</code></pre>

- `array` `&$reportsWithGoals` The list of arrays describing reports that have Goal metrics. Each element of this array must be an array with the following properties: - **category**: The report category. This should be a translated string. - **name**: The report's translated name. - **module**: The plugin the report is in, eg, `'UserCountry'`. - **action**: The API method of the report, eg, `'getCountry'`.

Usages:

[Goals::getActualReportsWithGoalMetrics](https://github.com/piwik/piwik/blob/master/plugins/Goals/Goals.php#L235)

## Insights

- [Insights.addReportToOverview](#insightsaddreporttooverview)

### Insights.addReportToOverview
_Defined in [Piwik/Plugins/Insights/API](https://github.com/piwik/piwik/blob/master/plugins/Insights/API.php) in line [69](https://github.com/piwik/piwik/blob/master/plugins/Insights/API.php#L69)_

Triggered to gather all reports to be displayed in the "Insight" and "Movers And Shakers" overview reports. Plugins that want to add new reports to the overview should subscribe to this event and add reports to the
incoming array. API parameters can be configured as an array optionally.

**Example**

    public function addReportToInsightsOverview(&$reports)
    {
        $reports['Actions_getPageUrls']  = array();
        $reports['Actions_getDownloads'] = array('flat' => 1, 'minGrowthPercent' => 60);
    }

Callback Signature:
<pre><code>function(&amp;$reports)</code></pre>

- `array` `&$reports` An array containing a report unique id as key and an array of API parameters as values.

Usages:

[Actions::addReportToInsightsOverview](https://github.com/piwik/piwik/blob/master/plugins/Actions/Actions.php#L86), [Referrers::addReportToInsightsOverview](https://github.com/piwik/piwik/blob/master/plugins/Referrers/Referrers.php#L35), [UserCountry::addReportToInsightsOverview](https://github.com/piwik/piwik/blob/master/plugins/UserCountry/UserCountry.php#L42)

## LanguageManager

- [LanguageManager.getAvailableLanguages](#languagemanagergetavailablelanguages)

### LanguageManager.getAvailableLanguages
_Defined in [Piwik/Plugins/LanguagesManager/API](https://github.com/piwik/piwik/blob/master/plugins/LanguagesManager/API.php) in line [75](https://github.com/piwik/piwik/blob/master/plugins/LanguagesManager/API.php#L75)_

Hook called after loading available language files. Use this hook to customise the list of languagesPath available in Piwik.

Callback Signature:
<pre><code>function(&amp;$languages)</code></pre>

- `array`

## Live

- [Live.API.getIdSitesString](#liveapigetidsitesstring)
- [Live.getExtraVisitorDetails](#livegetextravisitordetails)
- [Live.makeNewVisitorObject](#livemakenewvisitorobject)

### Live.API.getIdSitesString
_Defined in [Piwik/Plugins/Live/API](https://github.com/piwik/piwik/blob/master/plugins/Live/API.php) in line [734](https://github.com/piwik/piwik/blob/master/plugins/Live/API.php#L734)_



Callback Signature:
<pre><code>function(&amp;$idSites)</code></pre>


### Live.getExtraVisitorDetails
_Defined in [Piwik/Plugins/Live/API](https://github.com/piwik/piwik/blob/master/plugins/Live/API.php) in line [399](https://github.com/piwik/piwik/blob/master/plugins/Live/API.php#L399)_

Triggered in the Live.getVisitorProfile API method. Plugins can use this event
to discover and add extra data to visitor profiles.

For example, if an email address is found in a custom variable, a plugin could load the
gravatar for the email and add it to the visitor profile, causing it to display in the
visitor profile popup.

The following visitor profile elements can be set to augment the visitor profile popup:

- **visitorAvatar**: A URL to an image to display in the top left corner of the popup.
- **visitorDescription**: Text to be used as the tooltip of the avatar image.

Callback Signature:
<pre><code>function(&amp;$result)</code></pre>

- `array` `$visitorProfile` The unaugmented visitor profile info.


### Live.makeNewVisitorObject
_Defined in [Piwik/Plugins/Live/VisitorFactory](https://github.com/piwik/piwik/blob/master/plugins/Live/VisitorFactory.php) in line [39](https://github.com/piwik/piwik/blob/master/plugins/Live/VisitorFactory.php#L39)_

Triggered while visit is filtering in live plugin. Subscribers to this
event can force the use of a custom visitor object that extends from
Piwik\Plugins\Live\VisitorInterface.

Callback Signature:
<pre><code>function(&amp;$visitor, $visitorRawData)</code></pre>

- `\Piwik\Plugins\Live\VisitorInterface` `&$visitor` Initialized to null, but can be set to a new visitor object. If it isn't modified Piwik uses the default class.

- `array` `$visitorRawData` Raw data using in Visitor object constructor.

## Log

- [Log.formatDatabaseMessage](#logformatdatabasemessage)
- [Log.formatFileMessage](#logformatfilemessage)
- [Log.formatScreenMessage](#logformatscreenmessage)
- [Log.getAvailableWriters](#loggetavailablewriters)

### Log.formatDatabaseMessage
_Defined in [Piwik/Log](https://github.com/piwik/piwik/blob/master/core/Log.php) in line [637](https://github.com/piwik/piwik/blob/master/core/Log.php#L637)_

Triggered when trying to log an object to a database table. Plugins can use
this event to convert objects to strings before they are logged.

**Example**

    public function formatDatabaseMessage(&$message, $level, $tag, $datetime, $logger) {
        if ($message instanceof MyCustomDebugInfo) {
            $message = $message->formatForDatabase();
        }
    }

Callback Signature:
<pre><code>function(&amp;$message, $level, $tag, $datetime, $logger)</code></pre>

- `mixed` `&$message` The object that is being logged. Event handlers should check if the object is of a certain type and if it is, set `$message` to the string that should be logged.

- `int` `$level` The log level used with this log entry.

- `string` `$tag` The current plugin that started logging (or if no plugin, the current class).

- `string` `$datetime` Datetime of the logging call.

- `[Log](/api-reference/Piwik/Log)` `$logger` The Log singleton.


### Log.formatFileMessage
_Defined in [Piwik/Log](https://github.com/piwik/piwik/blob/master/core/Log.php) in line [677](https://github.com/piwik/piwik/blob/master/core/Log.php#L677)_

Triggered when trying to log an object to a file. Plugins can use
this event to convert objects to strings before they are logged.

**Example**

    public function formatFileMessage(&$message, $level, $tag, $datetime, $logger) {
        if ($message instanceof MyCustomDebugInfo) {
            $message = $message->formatForFile();
        }
    }

Callback Signature:
<pre><code>function(&amp;$message, $level, $tag, $datetime, $logger)</code></pre>

- `mixed` `&$message` The object that is being logged. Event handlers should check if the object is of a certain type and if it is, set `$message` to the string that should be logged.

- `int` `$level` The log level used with this log entry.

- `string` `$tag` The current plugin that started logging (or if no plugin, the current class).

- `string` `$datetime` Datetime of the logging call.

- `[Log](/api-reference/Piwik/Log)` `$logger` The Log singleton.


### Log.formatScreenMessage
_Defined in [Piwik/Log](https://github.com/piwik/piwik/blob/master/core/Log.php) in line [584](https://github.com/piwik/piwik/blob/master/core/Log.php#L584)_

Triggered when trying to log an object to the screen. Plugins can use
this event to convert objects to strings before they are logged.

The result of this callback can be HTML so no sanitization is done on the result.
This means **YOU MUST SANITIZE THE MESSAGE YOURSELF** if you use this event.

**Example**

    public function formatScreenMessage(&$message, $level, $tag, $datetime, $logger) {
        if ($message instanceof MyCustomDebugInfo) {
            $message = Common::sanitizeInputValue($message->formatForScreen());
        }
    }

Callback Signature:
<pre><code>function(&amp;$message, $level, $tag, $datetime, $logger)</code></pre>

- `mixed` `&$message` The object that is being logged. Event handlers should check if the object is of a certain type and if it is, set `$message` to the string that should be logged.

- `int` `$level` The log level used with this log entry.

- `string` `$tag` The current plugin that started logging (or if no plugin, the current class).

- `string` `$datetime` Datetime of the logging call.

- `[Log](/api-reference/Piwik/Log)` `$logger` The Log singleton.


### Log.getAvailableWriters
_Defined in [Piwik/Log](https://github.com/piwik/piwik/blob/master/core/Log.php) in line [355](https://github.com/piwik/piwik/blob/master/core/Log.php#L355)_

This event is called when the Log instance is created. Plugins can use this event to
make new logging writers available.

A logging writer is a callback with the following signature:

    function (int $level, string $tag, string $datetime, string $message)

`$level` is the log level to use, `$tag` is the log tag used, `$datetime` is the date time
of the logging call and `$message` is the formatted log message.

Logging writers must be associated by name in the array passed to event handlers. The
name specified can be used in Piwik's INI configuration.

**Example**

    public function getAvailableWriters(&$writers) {
        $writers['myloggername'] = function ($level, $tag, $datetime, $message) {
            // ...
        };
    }

    // 'myloggername' can now be used in the log_writers config option.

Callback Signature:
<pre><code>function(&amp;$writers)</code></pre>

- `array` `&$writers` Array mapping writer names with logging writers.

## Login

- [Login.authenticate](#loginauthenticate)
- [Login.authenticate.successful](#loginauthenticatesuccessful)
- [Login.initSession.end](#logininitsessionend)

### Login.authenticate
_Defined in [Piwik/Plugins/Login/Auth](https://github.com/piwik/piwik/blob/master/plugins/Login/Auth.php) in line [165](https://github.com/piwik/piwik/blob/master/plugins/Login/Auth.php#L165)_

Triggered before authenticate function. This event propagate login and token_auth which will be using in authenticate process.

This event exists to enable possibility for user authentication prevention.
For example when user is locked or inactive.

**Example**

    Piwik::addAction('Login.authenticate', function ($login, $tokenAuth) {
        if (!UserActivityManager::isActive ($login, $tokenAuth) {
            throw new Exception('Your account is inactive.');
        }
    });

Callback Signature:
<pre><code>function($login, $tokenAuth)</code></pre>

- `string` `$login` User login.

- `string` `$tokenAuth` User token auth.


### Login.authenticate.successful
_Defined in [Piwik/Plugins/Login/Auth](https://github.com/piwik/piwik/blob/master/plugins/Login/Auth.php) in line [227](https://github.com/piwik/piwik/blob/master/plugins/Login/Auth.php#L227)_

Triggered after successful authenticate, but before cookie creation. This event propagate login and token_auth which was used in authenticate process.

This event exists to enable the ability to custom action before the cookie will be created,
but after a successful authentication.
For example when user have to fill survey or change password.

**Example**

    Piwik::addAction('Login.authenticate.successful', function ($login, $tokenAuth) {
        // redirect to change password action
    });

Callback Signature:
<pre><code>function($login, $tokenAuth)</code></pre>

- `string` `$login` User login.

- `string` `$tokenAuth` User token auth.


### Login.initSession.end
_Defined in [Piwik/Plugins/Login/Auth](https://github.com/piwik/piwik/blob/master/plugins/Login/Auth.php) in line [99](https://github.com/piwik/piwik/blob/master/plugins/Login/Auth.php#L99)_

Triggered after session initialize. This event notify about end of init session process.

**Example**

    Piwik::addAction('Login.initSession.end', function () {
        // session has been initialized
    });

## Menu

- [Menu.Admin.addItems](#menuadminadditems)
- [Menu.Reporting.addItems](#menureportingadditems)
- [Menu.Top.addItems](#menutopadditems)

### Menu.Admin.addItems
_Defined in [Piwik/Menu/MenuAdmin](https://github.com/piwik/piwik/blob/master/core/Menu/MenuAdmin.php) in line [134](https://github.com/piwik/piwik/blob/master/core/Menu/MenuAdmin.php#L134)_



Callback Signature:
<pre><code>function()</code></pre>


### Menu.Reporting.addItems
_Defined in [Piwik/Menu/MenuReporting](https://github.com/piwik/piwik/blob/master/core/Menu/MenuReporting.php) in line [112](https://github.com/piwik/piwik/blob/master/core/Menu/MenuReporting.php#L112)_



Callback Signature:
<pre><code>function()</code></pre>


### Menu.Top.addItems
_Defined in [Piwik/Menu/MenuTop](https://github.com/piwik/piwik/blob/master/core/Menu/MenuTop.php) in line [101](https://github.com/piwik/piwik/blob/master/core/Menu/MenuTop.php#L101)_



Callback Signature:
<pre><code>function()</code></pre>

## Metrics

- [Metrics.getDefaultMetricDocumentationTranslations](#metricsgetdefaultmetricdocumentationtranslations)
- [Metrics.getDefaultMetricTranslations](#metricsgetdefaultmetrictranslations)

### Metrics.getDefaultMetricDocumentationTranslations
_Defined in [Piwik/Metrics](https://github.com/piwik/piwik/blob/master/core/Metrics.php) in line [379](https://github.com/piwik/piwik/blob/master/core/Metrics.php#L379)_

Use this event to register translations for metrics documentation processed by your plugin.

Callback Signature:
<pre><code>function(&amp;$translations)</code></pre>

- `string` `&$translations` The array mapping of column_name => Plugin_TranslationForColumnDocumentation

Usages:

[Actions::addMetricDocumentationTranslations](https://github.com/piwik/piwik/blob/master/plugins/Actions/Actions.php#L66), [Events::addMetricDocumentationTranslations](https://github.com/piwik/piwik/blob/master/plugins/Events/Events.php#L32)


### Metrics.getDefaultMetricTranslations
_Defined in [Piwik/Metrics](https://github.com/piwik/piwik/blob/master/core/Metrics.php) in line [271](https://github.com/piwik/piwik/blob/master/core/Metrics.php#L271)_

Use this event to register translations for metrics processed by your plugin.

Callback Signature:
<pre><code>function(&amp;$translations)</code></pre>

- `string` `&$translations` The array mapping of column_name => Plugin_TranslationForColumn

Usages:

[Actions::addMetricTranslations](https://github.com/piwik/piwik/blob/master/plugins/Actions/Actions.php#L43), [Events::addMetricTranslations](https://github.com/piwik/piwik/blob/master/plugins/Events/Events.php#L27), [Goals::addMetricTranslations](https://github.com/piwik/piwik/blob/master/plugins/Goals/Goals.php#L112), [MultiSites::addMetricTranslations](https://github.com/piwik/piwik/blob/master/plugins/MultiSites/MultiSites.php#L35), [UserSettings::addMetricTranslations](https://github.com/piwik/piwik/blob/master/plugins/UserSettings/UserSettings.php#L43), [VisitFrequency::addMetricTranslations](https://github.com/piwik/piwik/blob/master/plugins/VisitFrequency/VisitFrequency.php#L26)

## MobileMessaging

- [MobileMessaging.deletePhoneNumber](#mobilemessagingdeletephonenumber)

### MobileMessaging.deletePhoneNumber
_Defined in [Piwik/Plugins/MobileMessaging/API](https://github.com/piwik/piwik/blob/master/plugins/MobileMessaging/API.php) in line [221](https://github.com/piwik/piwik/blob/master/plugins/MobileMessaging/API.php#L221)_

Triggered after a phone number has been deleted. This event should be used to clean up any data that is
related to the now deleted phone number. The ScheduledReports plugin, for example, uses this event to remove
the phone number from all reports to make sure no text message will be sent to this phone number.

**Example**

    public function deletePhoneNumber($phoneNumber)
    {
        $this->unsubscribePhoneNumberFromScheduledReport($phoneNumber);
    }

Callback Signature:
<pre><code>function($phoneNumber)</code></pre>

- `string` `$phoneNumber` The phone number that was just deleted.

Usages:

[ScheduledReports::deletePhoneNumber](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/ScheduledReports.php#L390)

## Platform

- [Platform.initialized](#platforminitialized)

### Platform.initialized
_Defined in [Piwik/FrontController](https://github.com/piwik/piwik/blob/master/core/FrontController.php) in line [436](https://github.com/piwik/piwik/blob/master/core/FrontController.php#L436)_

Triggered after the platform is initialized and after the user has been authenticated, but before the platform has handled the request. Piwik uses this event to check for updates to Piwik.

Usages:

[CoreUpdater::updateCheck](https://github.com/piwik/piwik/blob/master/plugins/CoreUpdater/CoreUpdater.php#L149), [UsersManager::onPlatformInitialized](https://github.com/piwik/piwik/blob/master/plugins/UsersManager/UsersManager.php#L41)

## Provider

- [Provider.getCleanHostname](#providergetcleanhostname)

### Provider.getCleanHostname
_Defined in [Piwik/Plugins/Provider/Provider](https://github.com/piwik/piwik/blob/master/plugins/Provider/Provider.php) in line [96](https://github.com/piwik/piwik/blob/master/plugins/Provider/Provider.php#L96)_

Triggered when prettifying a hostname string. This event can be used to customize the way a hostname is displayed in the
Providers report.

**Example**

    public function getCleanHostname(&$cleanHostname, $hostname)
    {
        if ('fvae.VARG.ceaga.site.co.jp' == $hostname) {
            $cleanHostname = 'site.co.jp';
        }
    }

Callback Signature:
<pre><code>function(&amp;$cleanHostname, $hostname)</code></pre>

- `string` `&$cleanHostname` The hostname string to display. Set by the event handler.

- `string` `$hostname` The full hostname.

## Referrer

- [Referrer.addSearchEngineUrls](#referreraddsearchengineurls)
- [Referrer.addSocialUrls](#referreraddsocialurls)

### Referrer.addSearchEngineUrls
_Defined in [Piwik/Common](https://github.com/piwik/piwik/blob/master/core/Common.php) in line [770](https://github.com/piwik/piwik/blob/master/core/Common.php#L770)_



Callback Signature:
<pre><code>function(&amp;$searchEngines)</code></pre>


### Referrer.addSocialUrls
_Defined in [Piwik/Common](https://github.com/piwik/piwik/blob/master/core/Common.php) in line [809](https://github.com/piwik/piwik/blob/master/core/Common.php#L809)_



Callback Signature:
<pre><code>function(&amp;$socialUrls)</code></pre>

## Request

- [Request.dispatch](#requestdispatch)
- [Request.dispatch.end](#requestdispatchend)
- [Request.dispatchCoreAndPluginUpdatesScreen](#requestdispatchcoreandpluginupdatesscreen)
- [Request.initAuthenticationObject](#requestinitauthenticationobject)
- [Request.initAuthenticationObject](#requestinitauthenticationobject)
- [Request.initAuthenticationObject](#requestinitauthenticationobject)

### Request.dispatch
_Defined in [Piwik/FrontController](https://github.com/piwik/piwik/blob/master/core/FrontController.php) in line [559](https://github.com/piwik/piwik/blob/master/core/FrontController.php#L559)_

Triggered directly before controller actions are dispatched. This event can be used to modify the parameters passed to one or more controller actions
and can be used to change the controller action being dispatched to.

Callback Signature:
<pre><code>function(&amp;$module, &amp;$action, &amp;$parameters)</code></pre>

- `string` `&$module` The name of the plugin being dispatched to.

- `string` `&$action` The name of the controller method being dispatched to.

- `array` `&$parameters` The arguments passed to the controller action.

Usages:

[Installation::dispatchIfNotInstalledYet](https://github.com/piwik/piwik/blob/master/plugins/Installation/Installation.php#L48)


### Request.dispatch.end
_Defined in [Piwik/FrontController](https://github.com/piwik/piwik/blob/master/core/FrontController.php) in line [601](https://github.com/piwik/piwik/blob/master/core/FrontController.php#L601)_

Triggered after a controller action is successfully called. This event can be used to modify controller action output (if any) before the output is returned.

Callback Signature:
<pre><code>function(&amp;$result, $module, $action, $parameters)</code></pre>

- `mixed` `&$result` The controller action result.

- `array` `$parameters` The arguments passed to the controller action.


### Request.dispatchCoreAndPluginUpdatesScreen
_Defined in [Piwik/FrontController](https://github.com/piwik/piwik/blob/master/core/FrontController.php) in line [387](https://github.com/piwik/piwik/blob/master/core/FrontController.php#L387)_

Triggered just after the platform is initialized and plugins are loaded. This event can be used to do early initialization.

_Note: At this point the user is not authenticated yet._

Usages:

[CoreUpdater::dispatch](https://github.com/piwik/piwik/blob/master/plugins/CoreUpdater/CoreUpdater.php#L120)


### Request.initAuthenticationObject
_Defined in [Piwik/Tracker/Request](https://github.com/piwik/piwik/blob/master/core/Tracker/Request.php) in line [110](https://github.com/piwik/piwik/blob/master/core/Tracker/Request.php#L110)_



Usages:

[Login::initAuthenticationObject](https://github.com/piwik/piwik/blob/master/plugins/Login/Login.php#L75)


### Request.initAuthenticationObject
_Defined in [Piwik/Plugins/Overlay/API](https://github.com/piwik/piwik/blob/master/plugins/Overlay/API.php) in line [125](https://github.com/piwik/piwik/blob/master/plugins/Overlay/API.php#L125)_

Triggered immediately before the user is authenticated. This event can be used by plugins that provide their own authentication mechanism
to make that mechanism available. Subscribers should set the `'auth'` object in
the Piwik\Registry to an object that implements the Piwik\Auth interface.

**Example**

    use Piwik\Registry;

    public function initAuthenticationObject($activateCookieAuth)
    {
        Registry::set('auth', new LDAPAuth($activateCookieAuth));
    }

Callback Signature:
<pre><code>function($activateCookieAuth = true)</code></pre>

- `bool` `$activateCookieAuth` Whether authentication based on `$_COOKIE` values should be allowed.

Usages:

[Login::initAuthenticationObject](https://github.com/piwik/piwik/blob/master/plugins/Login/Login.php#L75)


### Request.initAuthenticationObject
_Defined in [Piwik/FrontController](https://github.com/piwik/piwik/blob/master/core/FrontController.php) in line [409](https://github.com/piwik/piwik/blob/master/core/FrontController.php#L409)_

Triggered before the user is authenticated, when the global authentication object should be created. Plugins that provide their own authentication implementation should use this event
to set the global authentication object (which must derive from Piwik\Auth).

**Example**

    Piwik::addAction('Request.initAuthenticationObject', function() {
        Piwik\Registry::set('auth', new MyAuthImplementation());
    });

Usages:

[Login::initAuthenticationObject](https://github.com/piwik/piwik/blob/master/plugins/Login/Login.php#L75)

## ScheduledReports

- [ScheduledReports.allowMultipleReports](#scheduledreportsallowmultiplereports)
- [ScheduledReports.getRendererInstance](#scheduledreportsgetrendererinstance)
- [ScheduledReports.getReportFormats](#scheduledreportsgetreportformats)
- [ScheduledReports.getReportMetadata](#scheduledreportsgetreportmetadata)
- [ScheduledReports.getReportParameters](#scheduledreportsgetreportparameters)
- [ScheduledReports.getReportRecipients](#scheduledreportsgetreportrecipients)
- [ScheduledReports.getReportTypes](#scheduledreportsgetreporttypes)
- [ScheduledReports.processReports](#scheduledreportsprocessreports)
- [ScheduledReports.sendReport](#scheduledreportssendreport)
- [ScheduledReports.validateReportParameters](#scheduledreportsvalidatereportparameters)

### ScheduledReports.allowMultipleReports
_Defined in [Piwik/Plugins/ScheduledReports/API](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/API.php) in line [776](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/API.php#L776)_

Triggered when we're determining if a scheduled report transport medium can handle sending multiple Piwik reports in one scheduled report or not. Plugins that provide their own transport mediums should use this
event to specify whether their backend can send more than one Piwik report
at a time.

Callback Signature:
<pre><code>function(&amp;$allowMultipleReports, $reportType)</code></pre>

- `bool` `&$allowMultipleReports` Whether the backend type can handle multiple Piwik reports or not.

- `string` `$reportType` A string ID describing how the report is sent, eg, `'sms'` or `'email'`.

Usages:

[MobileMessaging::allowMultipleReports](https://github.com/piwik/piwik/blob/master/plugins/MobileMessaging/MobileMessaging.php#L165), [ScheduledReports::allowMultipleReports](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/ScheduledReports.php#L253)


### ScheduledReports.getRendererInstance
_Defined in [Piwik/Plugins/ScheduledReports/API](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/API.php) in line [425](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/API.php#L425)_

Triggered when obtaining a renderer instance based on the scheduled report output format. Plugins that provide new scheduled report output formats should use this event to
handle their new report formats.

Callback Signature:
<pre><code>function(&amp;$reportRenderer, $reportType, $outputType, $report)</code></pre>

- `ReportRenderer` `&$reportRenderer` This variable should be set to an instance that extends Piwik\ReportRenderer by one of the event subscribers.

- `string` `$reportType` A string ID describing how the report is sent, eg, `'sms'` or `'email'`.

- `string` `$outputType` The output format of the report, eg, `'html'`, `'pdf'`, etc.

- `array` `&$report` An array describing the scheduled report that is being generated.

Usages:

[MobileMessaging::getRendererInstance](https://github.com/piwik/piwik/blob/master/plugins/MobileMessaging/MobileMessaging.php#L152), [ScheduledReports::getRendererInstance](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/ScheduledReports.php#L240)


### ScheduledReports.getReportFormats
_Defined in [Piwik/Plugins/ScheduledReports/API](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/API.php) in line [823](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/API.php#L823)_

Triggered when gathering all available scheduled report formats. Plugins that provide their own scheduled report format should use
this event to make their format available.

Callback Signature:
<pre><code>function(&amp;$reportFormats, $reportType)</code></pre>

- `array` `&$reportFormats` An array mapping string IDs for each available scheduled report format with icon paths for those formats. Add your new format's ID to this array.

- `string` `$reportType` A string ID describing how the report is sent, eg, `'sms'` or `'email'`.

Usages:

[MobileMessaging::getReportFormats](https://github.com/piwik/piwik/blob/master/plugins/MobileMessaging/MobileMessaging.php#L138), [ScheduledReports::getReportFormats](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/ScheduledReports.php#L187)


### ScheduledReports.getReportMetadata
_Defined in [Piwik/Plugins/ScheduledReports/API](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/API.php) in line [748](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/API.php#L748)_

TODO: change this event so it returns a list of API methods instead of report metadata arrays. Triggered when gathering the list of Piwik reports that can be used with a certain
transport medium.

Plugins that provide their own transport mediums should use this
event to list the Piwik reports that their backend supports.

Callback Signature:
<pre><code>function(&amp;$availableReportMetadata, $reportType, $idSite)</code></pre>

- `array` `&$availableReportMetadata` An array containg report metadata for each supported report.

- `string` `$reportType` A string ID describing how the report is sent, eg, `'sms'` or `'email'`.

- `int` `$idSite` The ID of the site we're getting available reports for.

Usages:

[MobileMessaging::getReportMetadata](https://github.com/piwik/piwik/blob/master/plugins/MobileMessaging/MobileMessaging.php#L115), [ScheduledReports::getReportMetadata](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/ScheduledReports.php#L162)


### ScheduledReports.getReportParameters
_Defined in [Piwik/Plugins/ScheduledReports/API](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/API.php) in line [602](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/API.php#L602)_

Triggered when gathering the available parameters for a scheduled report type. Plugins that provide their own scheduled report transport mediums should use this
event to list the available report parameters for their transport medium.

Callback Signature:
<pre><code>function(&amp;$availableParameters, $reportType)</code></pre>

- `array` `&$availableParameters` The list of available parameters for this report type. This is an array that maps paramater IDs with a boolean that indicates whether the parameter is mandatory or not.

- `string` `$reportType` A string ID describing how the report is sent, eg, `'sms'` or `'email'`.

Usages:

[MobileMessaging::getReportParameters](https://github.com/piwik/piwik/blob/master/plugins/MobileMessaging/MobileMessaging.php#L145), [ScheduledReports::getReportParameters](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/ScheduledReports.php#L194)


### ScheduledReports.getReportRecipients
_Defined in [Piwik/Plugins/ScheduledReports/API](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/API.php) in line [854](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/API.php#L854)_

Triggered when getting the list of recipients of a scheduled report. Plugins that provide their own scheduled report transport medium should use this event
to extract the list of recipients their backend's specific scheduled report
format.

Callback Signature:
<pre><code>function(&amp;$recipients, $report[&#039;type&#039;], $report)</code></pre>

- `array` `&$recipients` An array of strings describing each of the scheduled reports recipients. Can be, for example, a list of email addresses or phone numbers or whatever else your plugin uses.

- `string` `$reportType` A string ID describing how the report is sent, eg, `'sms'` or `'email'`.

- `array` `$report` An array describing the scheduled report that is being generated.

Usages:

[MobileMessaging::getReportRecipients](https://github.com/piwik/piwik/blob/master/plugins/MobileMessaging/MobileMessaging.php#L172), [ScheduledReports::getReportRecipients](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/ScheduledReports.php#L432)


### ScheduledReports.getReportTypes
_Defined in [Piwik/Plugins/ScheduledReports/API](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/API.php) in line [799](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/API.php#L799)_

Triggered when gathering all available transport mediums. Plugins that provide their own transport mediums should use this
event to make their medium available.

Callback Signature:
<pre><code>function(&amp;$reportTypes)</code></pre>

- `array` `&$reportTypes` An array mapping transport medium IDs with the paths to those mediums' icons. Add your new backend's ID to this array.

Usages:

[MobileMessaging::getReportTypes](https://github.com/piwik/piwik/blob/master/plugins/MobileMessaging/MobileMessaging.php#L133), [ScheduledReports::getReportTypes](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/ScheduledReports.php#L182)


### ScheduledReports.processReports
_Defined in [Piwik/Plugins/ScheduledReports/API](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/API.php) in line [403](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/API.php#L403)_

Triggered when generating the content of scheduled reports. This event can be used to modify the report data or report metadata of one or more reports
in a scheduled report, before the scheduled report is rendered and delivered.

TODO: list data available in $report or make it a new class that can be documented (same for
      all other events that use a $report)

Callback Signature:
<pre><code>function(&amp;$processedReports, $reportType, $outputType, $report)</code></pre>

- `array` `&$processedReports` The list of processed reports in the scheduled report. Entries includes report data and metadata for each report.

- `string` `$reportType` A string ID describing how the scheduled report will be sent, eg, `'sms'` or `'email'`.

- `string` `$outputType` The output format of the report, eg, `'html'`, `'pdf'`, etc.

- `array` `$report` An array describing the scheduled report that is being generated.

Usages:

[ScheduledReports::processReports](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/ScheduledReports.php#L201)


### ScheduledReports.sendReport
_Defined in [Piwik/Plugins/ScheduledReports/API](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/API.php) in line [544](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/API.php#L544)_

Triggered when sending scheduled reports. Plugins that provide new scheduled report transport mediums should use this event to
send the scheduled report.

Callback Signature:
<pre><code>function($report[&#039;type&#039;], $report, $contents, $filename, $prettyDate, $reportSubject, $reportTitle, $additionalFiles)</code></pre>

- `string` `$reportType` A string ID describing how the report is sent, eg, `'sms'` or `'email'`.

- `array` `$report` An array describing the scheduled report that is being generated.

- `string` `$contents` The contents of the scheduled report that was generated and now should be sent.

- `string` `$filename` The path to the file where the scheduled report has been saved.

- `string` `$prettyDate` A prettified date string for the data within the scheduled report.

- `string` `$reportSubject` A string describing what's in the scheduled report.

- `string` `$reportTitle` The scheduled report's given title (given by a Piwik user).

- `array` `$additionalFiles` The list of additional files that should be sent with this report.

Usages:

[MobileMessaging::sendReport](https://github.com/piwik/piwik/blob/master/plugins/MobileMessaging/MobileMessaging.php#L179), [ScheduledReports::sendReport](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/ScheduledReports.php#L260)


### ScheduledReports.validateReportParameters
_Defined in [Piwik/Plugins/ScheduledReports/API](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/API.php) in line [629](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/API.php#L629)_

Triggered when validating the parameters for a scheduled report. Plugins that provide their own scheduled reports backend should use this
event to validate the custom parameters defined with ScheduledReports::getReportParameters().

Callback Signature:
<pre><code>function(&amp;$parameters, $reportType)</code></pre>

- `array` `&$parameters` The list of parameters for the scheduled report.

- `string` `$reportType` A string ID describing how the report is sent, eg, `'sms'` or `'email'`.

Usages:

[MobileMessaging::validateReportParameters](https://github.com/piwik/piwik/blob/master/plugins/MobileMessaging/MobileMessaging.php#L96), [ScheduledReports::validateReportParameters](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/ScheduledReports.php#L120)

## SegmentEditor

- [SegmentEditor.deactivate](#segmenteditordeactivate)
- [SegmentEditor.update](#segmenteditorupdate)

### SegmentEditor.deactivate
_Defined in [Piwik/Plugins/SegmentEditor/API](https://github.com/piwik/piwik/blob/master/plugins/SegmentEditor/API.php) in line [172](https://github.com/piwik/piwik/blob/master/plugins/SegmentEditor/API.php#L172)_

Triggered before a segment is deleted or made invisible. This event can be used by plugins to throw an exception
or do something else.

Callback Signature:
<pre><code>function($idSegment)</code></pre>

- `int` `$idSegment` The ID of the segment being deleted.

Usages:

[ScheduledReports::segmentDeactivation](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/ScheduledReports.php#L500)


### SegmentEditor.update
_Defined in [Piwik/Plugins/SegmentEditor/API](https://github.com/piwik/piwik/blob/master/plugins/SegmentEditor/API.php) in line [219](https://github.com/piwik/piwik/blob/master/plugins/SegmentEditor/API.php#L219)_

Triggered before a segment is modified. This event can be used by plugins to throw an exception
or do something else.

Callback Signature:
<pre><code>function($idSegment, $bind)</code></pre>

- `int` `$idSegment` The ID of the segment which visibility is reduced.

Usages:

[ScheduledReports::segmentUpdated](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/ScheduledReports.php#L466)

## Segments

- [Segments.getKnownSegmentsToArchiveAllSites](#segmentsgetknownsegmentstoarchiveallsites)
- [Segments.getKnownSegmentsToArchiveForSite](#segmentsgetknownsegmentstoarchiveforsite)

### Segments.getKnownSegmentsToArchiveAllSites
_Defined in [Piwik/SettingsPiwik](https://github.com/piwik/piwik/blob/master/core/SettingsPiwik.php) in line [89](https://github.com/piwik/piwik/blob/master/core/SettingsPiwik.php#L89)_

Triggered during the cron archiving process to collect segments that should be pre-processed for all websites. The archiving process will be launched
for each of these segments when archiving data.

This event can be used to add segments to be pre-processed. If your plugin depends
on data from a specific segment, this event could be used to provide enhanced
performance.

_Note: If you just want to add a segment that is managed by the user, use the
SegmentEditor API._

**Example**

    Piwik::addAction('Segments.getKnownSegmentsToArchiveAllSites', function (&$segments) {
        $segments[] = 'country=jp;city=Tokyo';
    });

Callback Signature:
<pre><code>function(&amp;$segmentsToProcess)</code></pre>

- `array` `&$segmentsToProcess` List of segment definitions, eg, array( 'browserCode=ff;resolution=800x600', 'country=jp;city=Tokyo' ) Add segments to this array in your event handler.

Usages:

[SegmentEditor::getKnownSegmentsToArchiveAllSites](https://github.com/piwik/piwik/blob/master/plugins/SegmentEditor/SegmentEditor.php#L53)


### Segments.getKnownSegmentsToArchiveForSite
_Defined in [Piwik/SettingsPiwik](https://github.com/piwik/piwik/blob/master/core/SettingsPiwik.php) in line [134](https://github.com/piwik/piwik/blob/master/core/SettingsPiwik.php#L134)_

Triggered during the cron archiving process to collect segments that should be pre-processed for one specific site. The archiving process will be launched
for each of these segments when archiving data for that one site.

This event can be used to add segments to be pre-processed for one site.

_Note: If you just want to add a segment that is managed by the user, you should use the
SegmentEditor API._

**Example**

    Piwik::addAction('Segments.getKnownSegmentsToArchiveForSite', function (&$segments, $idSite) {
        $segments[] = 'country=jp;city=Tokyo';
    });

Callback Signature:
<pre><code>function(&amp;$segments, $idSite)</code></pre>

- `array` `$segmentsToProcess` List of segment definitions, eg, array( 'browserCode=ff;resolution=800x600', 'country=JP;city=Tokyo' ) Add segments to this array in your event handler.

- `int` `$idSite` The ID of the site to get segments for.

Usages:

[SegmentEditor::getKnownSegmentsToArchiveForSite](https://github.com/piwik/piwik/blob/master/plugins/SegmentEditor/SegmentEditor.php#L65)

## Site

- [Site.setSite](#sitesetsite)

### Site.setSite
_Defined in [Piwik/Site](https://github.com/piwik/piwik/blob/master/core/Site.php) in line [117](https://github.com/piwik/piwik/blob/master/core/Site.php#L117)_

Triggered so plugins can modify website entities without modifying the database. This event should **not** be used to add data that is expensive to compute. If you
need to make HTTP requests or query the database for more information, this is not
the place to do it.

**Example**

    Piwik::addAction('Site.setSite', function ($idSite, &$info) {
        $info['name'] .= " (original)";
    });

Callback Signature:
<pre><code>function($idSite, &amp;$infoSite)</code></pre>

- `int` `$idSite` The ID of the website entity that will be modified.

- `array` `&$infoSite` The website entity. [Learn more.](/guides/persistence-and-the-mysql-backend#websites-aka-sites)

## SitesManager

- [SitesManager.addSite.end](#sitesmanageraddsiteend)
- [SitesManager.deleteSite.end](#sitesmanagerdeletesiteend)
- [SitesManager.getImageTrackingCode](#sitesmanagergetimagetrackingcode)

### SitesManager.addSite.end
_Defined in [Piwik/Plugins/SitesManager/API](https://github.com/piwik/piwik/blob/master/plugins/SitesManager/API.php) in line [621](https://github.com/piwik/piwik/blob/master/plugins/SitesManager/API.php#L621)_

Triggered after a site has been added.

Callback Signature:
<pre><code>function($idSite)</code></pre>

- `int` `$idSite` The ID of the site that was added.


### SitesManager.deleteSite.end
_Defined in [Piwik/Plugins/SitesManager/API](https://github.com/piwik/piwik/blob/master/plugins/SitesManager/API.php) in line [676](https://github.com/piwik/piwik/blob/master/plugins/SitesManager/API.php#L676)_

Triggered after a site has been deleted. Plugins can use this event to remove site specific values or settings, such as removing all
goals that belong to a specific website. If you store any data related to a website you
should clean up that information here.

Callback Signature:
<pre><code>function($idSite)</code></pre>

- `int` `$idSite` The ID of the site being deleted.

Usages:

[Goals::deleteSiteGoals](https://github.com/piwik/piwik/blob/master/plugins/Goals/Goals.php#L136), [ScheduledReports::deleteSiteReport](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/ScheduledReports.php#L105), [UsersManager::deleteSite](https://github.com/piwik/piwik/blob/master/plugins/UsersManager/UsersManager.php#L71)


### SitesManager.getImageTrackingCode
_Defined in [Piwik/Plugins/SitesManager/API](https://github.com/piwik/piwik/blob/master/plugins/SitesManager/API.php) in line [129](https://github.com/piwik/piwik/blob/master/plugins/SitesManager/API.php#L129)_

Triggered when generating image link tracking code server side. Plugins can use
this event to customise the image tracking code that is displayed to the
user.

Callback Signature:
<pre><code>function(&amp;$piwikUrl, &amp;$urlParams)</code></pre>

- `string` `$piwikHost` The domain and URL path to the Piwik installation, eg, `'examplepiwik.com/path/to/piwik'`.

- `array` `&$urlParams` The query parameters used in the <img> element's src URL. See Piwik's image tracking docs for more info.

## TaskScheduler

- [TaskScheduler.getScheduledTasks](#taskschedulergetscheduledtasks)

### TaskScheduler.getScheduledTasks
_Defined in [Piwik/TaskScheduler](https://github.com/piwik/piwik/blob/master/core/TaskScheduler.php) in line [94](https://github.com/piwik/piwik/blob/master/core/TaskScheduler.php#L94)_



Callback Signature:
<pre><code>function(&amp;$tasks)</code></pre>

## Tracker

- [Tracker.Cache.getSiteAttributes](#trackercachegetsiteattributes)
- [Tracker.detectReferrerSearchEngine](#trackerdetectreferrersearchengine)
- [Tracker.end](#trackerend)
- [Tracker.existingVisitInformation](#trackerexistingvisitinformation)
- [Tracker.getDatabaseConfig](#trackergetdatabaseconfig)
- [Tracker.isExcludedVisit](#trackerisexcludedvisit)
- [Tracker.makeNewVisitObject](#trackermakenewvisitobject)
- [Tracker.newConversionInformation](#trackernewconversioninformation)
- [Tracker.newVisitorInformation](#trackernewvisitorinformation)
- [Tracker.recordAction](#trackerrecordaction)
- [Tracker.recordEcommerceGoal](#trackerrecordecommercegoal)
- [Tracker.recordStandardGoals](#trackerrecordstandardgoals)
- [Tracker.Request.getIdSite](#trackerrequestgetidsite)
- [Tracker.setTrackerCacheGeneral](#trackersettrackercachegeneral)
- [Tracker.setVisitorIp](#trackersetvisitorip)

### Tracker.Cache.getSiteAttributes
_Defined in [Piwik/Tracker/Cache](https://github.com/piwik/piwik/blob/master/core/Tracker/Cache.php) in line [87](https://github.com/piwik/piwik/blob/master/core/Tracker/Cache.php#L87)_

Triggered to get the attributes of a site entity that might be used by the Tracker. Plugins add new site attributes for use in other tracking events must
use this event to put those attributes in the Tracker Cache.

**Example**

    public function getSiteAttributes($content, $idSite)
    {
        $sql = "SELECT info FROM " . Common::prefixTable('myplugin_extra_site_info') . " WHERE idsite = ?";
        $content['myplugin_site_data'] = Db::fetchOne($sql, array($idSite));
    }

Callback Signature:
<pre><code>function(&amp;$content, $idSite)</code></pre>

- `array` `&$content` Array mapping of site attribute names with values.

- `int` `$idSite` The site ID to get attributes for.

Usages:

[Goals::fetchGoalsFromDb](https://github.com/piwik/piwik/blob/master/plugins/Goals/Goals.php#L264), [SitesManager::recordWebsiteDataInCache](https://github.com/piwik/piwik/blob/master/plugins/SitesManager/SitesManager.php#L61), [UsersManager::recordAdminUsersInCache](https://github.com/piwik/piwik/blob/master/plugins/UsersManager/UsersManager.php#L56)


### Tracker.detectReferrerSearchEngine
_Defined in [Piwik/Plugins/Referrers/Columns/Base](https://github.com/piwik/piwik/blob/master/plugins/Referrers/Columns/Base.php) in line [154](https://github.com/piwik/piwik/blob/master/plugins/Referrers/Columns/Base.php#L154)_

Triggered when detecting the search engine of a referrer URL. Plugins can use this event to provide custom search engine detection
logic.

Callback Signature:
<pre><code>function(&amp;$searchEngineInformation, $this-&gt;referrerUrl)</code></pre>

- `array` `&$searchEngineInformation` An array with the following information: - **name**: The search engine name. - **keywords**: The search keywords used. This parameter is initialized to the results of Piwik's default search engine detection logic.

- `string`


### Tracker.end
_Defined in [Piwik/Tracker](https://github.com/piwik/piwik/blob/master/core/Tracker.php) in line [256](https://github.com/piwik/piwik/blob/master/core/Tracker.php#L256)_




### Tracker.existingVisitInformation
_Defined in [Piwik/Tracker/Visit](https://github.com/piwik/piwik/blob/master/core/Tracker/Visit.php) in line [260](https://github.com/piwik/piwik/blob/master/core/Tracker/Visit.php#L260)_

Triggered before a [visit entity](/guides/persistence-and-the-mysql-backend#visits) is updated when tracking an action for an existing visit. This event can be used to modify the visit properties that will be updated before the changes
are persisted.

Callback Signature:
<pre><code>function(&amp;$valuesToUpdate, $this-&gt;visitorInfo)</code></pre>

- `array` `&$valuesToUpdate` Visit entity properties that will be updated.

- `array` `$visit` The entire visit entity. Read [this](/guides/persistence-and-the-mysql-backend#visits) to see what it contains.


### Tracker.getDatabaseConfig
_Defined in [Piwik/Tracker](https://github.com/piwik/piwik/blob/master/core/Tracker.php) in line [565](https://github.com/piwik/piwik/blob/master/core/Tracker.php#L565)_

Triggered before a connection to the database is established by the Tracker. This event can be used to change the database connection settings used by the Tracker.

Callback Signature:
<pre><code>function(&amp;$configDb)</code></pre>

- `array` `$dbInfos` Reference to an array containing database connection info, including: - **host**: The host name or IP address to the MySQL database. - **username**: The username to use when connecting to the database. - **password**: The password to use when connecting to the database. - **dbname**: The name of the Piwik MySQL database. - **port**: The MySQL database port to use. - **adapter**: either `'PDO\MYSQL'` or `'MYSQLI'` - **type**: The MySQL engine to use, for instance 'InnoDB'


### Tracker.isExcludedVisit
_Defined in [Piwik/Tracker/VisitExcluded](https://github.com/piwik/piwik/blob/master/core/Tracker/VisitExcluded.php) in line [85](https://github.com/piwik/piwik/blob/master/core/Tracker/VisitExcluded.php#L85)_

Triggered on every tracking request. This event can be used to tell the Tracker not to record this particular action or visit.

Callback Signature:
<pre><code>function(&amp;$excluded)</code></pre>

- `bool` `&$excluded` Whether the request should be excluded or not. Initialized to `false`. Event subscribers should set it to `true` in order to exclude the request.


### Tracker.makeNewVisitObject
_Defined in [Piwik/Tracker](https://github.com/piwik/piwik/blob/master/core/Tracker.php) in line [647](https://github.com/piwik/piwik/blob/master/core/Tracker.php#L647)_

Triggered before a new **visit tracking object** is created. Subscribers to this
event can force the use of a custom visit tracking object that extends from
Piwik\Tracker\VisitInterface.

Callback Signature:
<pre><code>function(&amp;$visit)</code></pre>

- `\Piwik\Tracker\VisitInterface` `&$visit` Initialized to null, but can be set to a new visit object. If it isn't modified Piwik uses the default class.


### Tracker.newConversionInformation
_Defined in [Piwik/Tracker/GoalManager](https://github.com/piwik/piwik/blob/master/core/Tracker/GoalManager.php) in line [724](https://github.com/piwik/piwik/blob/master/core/Tracker/GoalManager.php#L724)_

Triggered before persisting a new [conversion entity](/guides/persistence-and-the-mysql-backend#conversions). This event can be used to modify conversion information or to add new information to be persisted.

Callback Signature:
<pre><code>function(&amp;$conversion, $visitInformation, $this-&gt;request)</code></pre>

- `array` `&$conversion` The conversion entity. Read [this](/guides/persistence-and-the-mysql-backend#conversions) to see what it contains.

- `array` `$visitInformation` The visit entity that we are tracking a conversion for. See what information it contains [here](/guides/persistence-and-the-mysql-backend#visits).

- `\Piwik\Tracker\Request` `$request` An object describing the tracking request being processed.


### Tracker.newVisitorInformation
_Defined in [Piwik/Tracker/Visit](https://github.com/piwik/piwik/blob/master/core/Tracker/Visit.php) in line [327](https://github.com/piwik/piwik/blob/master/core/Tracker/Visit.php#L327)_

Triggered before a new [visit entity](/guides/persistence-and-the-mysql-backend#visits) is persisted. This event can be used to modify the visit entity or add new information to it before it is persisted.
The UserCountry plugin, for example, uses this event to add location information for each visit.

Callback Signature:
<pre><code>function(&amp;$this-&gt;visitorInfo, $this-&gt;request)</code></pre>

- `array` `$visit` The visit entity. Read [this](/guides/persistence-and-the-mysql-backend#visits) to see what information it contains.

- `\Piwik\Tracker\Request` `$request` An object describing the tracking request being processed.


### Tracker.recordAction
_Defined in [Piwik/Tracker/Action](https://github.com/piwik/piwik/blob/master/core/Tracker/Action.php) in line [364](https://github.com/piwik/piwik/blob/master/core/Tracker/Action.php#L364)_

Triggered after successfully persisting a [visit action entity](/guides/persistence-and-the-mysql-backend#visit-actions).

Callback Signature:
<pre><code>function($trackerAction = $this, $visitAction)</code></pre>

- `Action` `$tracker` Action The Action tracker instance.

- `array` `$visitAction` The visit action entity that was persisted. Read [this](/guides/persistence-and-the-mysql-backend#visit-actions) to see what it contains.


### Tracker.recordEcommerceGoal
_Defined in [Piwik/Tracker/GoalManager](https://github.com/piwik/piwik/blob/master/core/Tracker/GoalManager.php) in line [317](https://github.com/piwik/piwik/blob/master/core/Tracker/GoalManager.php#L317)_

Triggered after successfully persisting an ecommerce conversion. _Note: Subscribers should be wary of doing any expensive computation here as it may slow
the tracker down._

Callback Signature:
<pre><code>function($conversion, $visitInformation)</code></pre>

- `array` `$conversion` The conversion entity that was just persisted. See what information it contains [here](/guides/persistence-and-the-mysql-backend#conversions).

- `array` `$visitInformation` The visit entity that we are tracking a conversion for. See what information it contains [here](/guides/persistence-and-the-mysql-backend#visits).


### Tracker.recordStandardGoals
_Defined in [Piwik/Tracker/GoalManager](https://github.com/piwik/piwik/blob/master/core/Tracker/GoalManager.php) in line [700](https://github.com/piwik/piwik/blob/master/core/Tracker/GoalManager.php#L700)_

Triggered after successfully recording a non-ecommerce conversion. _Note: Subscribers should be wary of doing any expensive computation here as it may slow
the tracker down._

Callback Signature:
<pre><code>function($conversion)</code></pre>

- `array` `$conversion` The conversion entity that was just persisted. See what information it contains [here](/guides/persistence-and-the-mysql-backend#conversions).


### Tracker.Request.getIdSite
_Defined in [Piwik/Tracker/Request](https://github.com/piwik/piwik/blob/master/core/Tracker/Request.php) in line [335](https://github.com/piwik/piwik/blob/master/core/Tracker/Request.php#L335)_

Triggered when obtaining the ID of the site we are tracking a visit for. This event can be used to change the site ID so data is tracked for a different
website.

Callback Signature:
<pre><code>function(&amp;$idSite, $this-&gt;params)</code></pre>

- `int` `&$idSite` Initialized to the value of the **idsite** query parameter. If a subscriber sets this variable, the value it uses must be greater than 0.

- `array` `$params` The entire array of request parameters in the current tracking request.


### Tracker.setTrackerCacheGeneral
_Defined in [Piwik/Tracker/Cache](https://github.com/piwik/piwik/blob/master/core/Tracker/Cache.php) in line [150](https://github.com/piwik/piwik/blob/master/core/Tracker/Cache.php#L150)_

Triggered before the [general tracker cache](/guides/all-about-tracking#the-tracker-cache) is saved to disk. This event can be used to add extra content to the cache.

Data that is used during tracking but is expensive to compute/query should be
cached to keep tracking efficient. One example of such data are options
that are stored in the piwik_option table. Querying data for each tracking
request means an extra unnecessary database query for each visitor action. Using
a cache solves this problem.

**Example**

    public function setTrackerCacheGeneral(&$cacheContent)
    {
        $cacheContent['MyPlugin.myCacheKey'] = Option::get('MyPlugin_myOption');
    }

Callback Signature:
<pre><code>function(&amp;$cacheContent)</code></pre>

- `array` `&$cacheContent` Array of cached data. Each piece of data must be mapped by name.

Usages:

[PrivacyManager::setTrackerCacheGeneral](https://github.com/piwik/piwik/blob/master/plugins/PrivacyManager/PrivacyManager.php#L143), [UserCountry::setTrackerCacheGeneral](https://github.com/piwik/piwik/blob/master/plugins/UserCountry/UserCountry.php#L47)


### Tracker.setVisitorIp
_Defined in [Piwik/Tracker/Visit](https://github.com/piwik/piwik/blob/master/core/Tracker/Visit.php) in line [99](https://github.com/piwik/piwik/blob/master/core/Tracker/Visit.php#L99)_

Triggered after visits are tested for exclusion so plugins can modify the IP address persisted with a visit. This event is primarily used by the **PrivacyManager** plugin to anonymize IP addresses.

Callback Signature:
<pre><code>function(&amp;$this-&gt;visitorInfo[&#039;location_ip&#039;])</code></pre>

- `string` `$ip` The visitor's IP address.

## Translate

- [Translate.getClientSideTranslationKeys](#translategetclientsidetranslationkeys)

### Translate.getClientSideTranslationKeys
_Defined in [Piwik/Translate](https://github.com/piwik/piwik/blob/master/core/Translate.php) in line [208](https://github.com/piwik/piwik/blob/master/core/Translate.php#L208)_

Triggered before generating the JavaScript code that allows i18n strings to be used in the browser. Plugins should subscribe to this event to specify which translations
should be available to JavaScript.

Event handlers should add whole translation keys, ie, keys that include the plugin name.

**Example**

    public function getClientSideTranslationKeys(&$result)
    {
        $result[] = "MyPlugin_MyTranslation";
    }

Callback Signature:
<pre><code>function(&amp;$result)</code></pre>

- `array` `&$result` The whole list of client side translation keys.

Usages:

[CoreHome::getClientSideTranslationKeys](https://github.com/piwik/piwik/blob/master/plugins/CoreHome/CoreHome.php#L125), [CorePluginsAdmin::getClientSideTranslationKeys](https://github.com/piwik/piwik/blob/master/plugins/CorePluginsAdmin/CorePluginsAdmin.php#L53), [CoreVisualizations::getClientSideTranslationKeys](https://github.com/piwik/piwik/blob/master/plugins/CoreVisualizations/CoreVisualizations.php#L67), [Dashboard::getClientSideTranslationKeys](https://github.com/piwik/piwik/blob/master/plugins/Dashboard/Dashboard.php#L241), [Feedback::getClientSideTranslationKeys](https://github.com/piwik/piwik/blob/master/plugins/Feedback/Feedback.php#L43), [Goals::getClientSideTranslationKeys](https://github.com/piwik/piwik/blob/master/plugins/Goals/Goals.php#L270), [Live::getClientSideTranslationKeys](https://github.com/piwik/piwik/blob/master/plugins/Live/Live.php#L46), [MultiSites::getClientSideTranslationKeys](https://github.com/piwik/piwik/blob/master/plugins/MultiSites/MultiSites.php#L51), [Overlay::getClientSideTranslationKeys](https://github.com/piwik/piwik/blob/master/plugins/Overlay/Overlay.php#L43), [ScheduledReports::getClientSideTranslationKeys](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/ScheduledReports.php#L96), [SitesManager::getClientSideTranslationKeys](https://github.com/piwik/piwik/blob/master/plugins/SitesManager/SitesManager.php#L200), [Transitions::getClientSideTranslationKeys](https://github.com/piwik/piwik/blob/master/plugins/Transitions/Transitions.php#L38), [UserCountry::getClientSideTranslationKeys](https://github.com/piwik/piwik/blob/master/plugins/UserCountry/UserCountry.php#L94), [UserCountryMap::getClientSideTranslationKeys](https://github.com/piwik/piwik/blob/master/plugins/UserCountryMap/UserCountryMap.php#L75), [UsersManager::getClientSideTranslationKeys](https://github.com/piwik/piwik/blob/master/plugins/UsersManager/UsersManager.php#L145), [Widgetize::getClientSideTranslationKeys](https://github.com/piwik/piwik/blob/master/plugins/Widgetize/Widgetize.php#L48), [ZenMode::getClientSideTranslationKeys](https://github.com/piwik/piwik/blob/master/plugins/ZenMode/ZenMode.php#L27)

## User

- [User.getLanguage](#usergetlanguage)
- [User.isNotAuthorized](#userisnotauthorized)

### User.getLanguage
_Defined in [Piwik/Translate](https://github.com/piwik/piwik/blob/master/core/Translate.php) in line [127](https://github.com/piwik/piwik/blob/master/core/Translate.php#L127)_

Triggered when the current user's language is requested. By default the current language is determined by the **language** query
parameter. Plugins can override this logic by subscribing to this event.

**Example**

    public function getLanguage(&$lang)
    {
        $client = new My3rdPartyAPIClient();
        $thirdPartyLang = $client->getLanguageForUser(Piwik::getCurrentUserLogin());

        if (!empty($thirdPartyLang)) {
            $lang = $thirdPartyLang;
        }
    }

Callback Signature:
<pre><code>function(&amp;$lang)</code></pre>

- `string` `&$lang` The language that should be used for the current user. Will be initialized to the value of the **language** query parameter.

Usages:

[LanguagesManager::getLanguageToLoad](https://github.com/piwik/piwik/blob/master/plugins/LanguagesManager/LanguagesManager.php#L93)


### User.isNotAuthorized
_Defined in [Piwik/FrontController](https://github.com/piwik/piwik/blob/master/core/FrontController.php) in line [99](https://github.com/piwik/piwik/blob/master/core/FrontController.php#L99)_

Triggered when a user with insufficient access permissions tries to view some resource. This event can be used to customize the error that occurs when a user is denied access
(for example, displaying an error message, redirecting to a page other than login, etc.).

Callback Signature:
<pre><code>function($exception)</code></pre>

- `[NoAccessException](/api-reference/Piwik/NoAccessException)` `$exception` The exception that was caught.

Usages:

[Login::noAccess](https://github.com/piwik/piwik/blob/master/plugins/Login/Login.php#L48)

## UsersManager

- [UsersManager.addUser.end](#usersmanageradduserend)
- [UsersManager.checkPassword](#usersmanagercheckpassword)
- [UsersManager.deleteUser](#usersmanagerdeleteuser)
- [UsersManager.getDefaultDates](#usersmanagergetdefaultdates)
- [UsersManager.removeSiteAccess](#usersmanagerremovesiteaccess)
- [UsersManager.removeSiteAccess](#usersmanagerremovesiteaccess)
- [UsersManager.updateUser.end](#usersmanagerupdateuserend)

### UsersManager.addUser.end
_Defined in [Piwik/Plugins/UsersManager/API](https://github.com/piwik/piwik/blob/master/plugins/UsersManager/API.php) in line [347](https://github.com/piwik/piwik/blob/master/plugins/UsersManager/API.php#L347)_

Triggered after a new user is created.

Callback Signature:
<pre><code>function($userLogin, $email, $password, $alias)</code></pre>

- `string` `$userLogin` The new user's login handle.


### UsersManager.checkPassword
_Defined in [Piwik/Plugins/UsersManager/UsersManager](https://github.com/piwik/piwik/blob/master/plugins/UsersManager/UsersManager.php) in line [130](https://github.com/piwik/piwik/blob/master/plugins/UsersManager/UsersManager.php#L130)_

Triggered before core password validator check password. This event exists for enable option to create custom password validation rules.
It can be used to validate password (length, used chars etc) and to notify about checking password.

**Example**

    Piwik::addAction('UsersManager.checkPassword', function ($password) {
        if (strlen($password) < 10) {
            throw new Exception('Password is too short.');
        }
    });

Callback Signature:
<pre><code>function($password)</code></pre>

- `string` `$password` Checking password in plain text.


### UsersManager.deleteUser
_Defined in [Piwik/Plugins/UsersManager/Model](https://github.com/piwik/piwik/blob/master/plugins/UsersManager/Model.php) in line [255](https://github.com/piwik/piwik/blob/master/plugins/UsersManager/Model.php#L255)_

Triggered after a user has been deleted. This event should be used to clean up any data that is related to the now deleted user.
The **Dashboard** plugin, for example, uses this event to remove the user's dashboards.

Callback Signature:
<pre><code>function($userLogin)</code></pre>

- `string` `$userLogin` The login handle of the deleted user.

Usages:

[CoreAdminHome::cleanupUser](https://github.com/piwik/piwik/blob/master/plugins/CoreAdminHome/CoreAdminHome.php#L31), [CoreVisualizations::deleteUser](https://github.com/piwik/piwik/blob/master/plugins/CoreVisualizations/CoreVisualizations.php#L39), [Dashboard::deleteDashboardLayout](https://github.com/piwik/piwik/blob/master/plugins/Dashboard/Dashboard.php#L220), [LanguagesManager::deleteUserLanguage](https://github.com/piwik/piwik/blob/master/plugins/LanguagesManager/LanguagesManager.php#L103), [ScheduledReports::deleteUserReport](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/ScheduledReports.php#L527)


### UsersManager.getDefaultDates
_Defined in [Piwik/Plugins/UsersManager/Controller](https://github.com/piwik/piwik/blob/master/plugins/UsersManager/Controller.php) in line [194](https://github.com/piwik/piwik/blob/master/plugins/UsersManager/Controller.php#L194)_

Triggered when the list of available dates is requested, for example for the User Settings > Report date to load by default.

Callback Signature:
<pre><code>function(&amp;$dates)</code></pre>

- `array` `&$dates` Array of (date => translation)


### UsersManager.removeSiteAccess
_Defined in [Piwik/Plugins/ScheduledReports/tests/ScheduledReportsTest](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/tests/ScheduledReportsTest.php) in line [94](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/tests/ScheduledReportsTest.php#L94)_



Callback Signature:
<pre><code>function(&#039;userLogin&#039;, function(1, 2))</code></pre>

Usages:

[ScheduledReports::deleteUserReportForSites](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/ScheduledReports.php#L532)


### UsersManager.removeSiteAccess
_Defined in [Piwik/Plugins/UsersManager/API](https://github.com/piwik/piwik/blob/master/plugins/UsersManager/API.php) in line [575](https://github.com/piwik/piwik/blob/master/plugins/UsersManager/API.php#L575)_



Callback Signature:
<pre><code>function($userLogin, $idSites)</code></pre>

Usages:

[ScheduledReports::deleteUserReportForSites](https://github.com/piwik/piwik/blob/master/plugins/ScheduledReports/ScheduledReports.php#L532)


### UsersManager.updateUser.end
_Defined in [Piwik/Plugins/UsersManager/API](https://github.com/piwik/piwik/blob/master/plugins/UsersManager/API.php) in line [452](https://github.com/piwik/piwik/blob/master/plugins/UsersManager/API.php#L452)_

Triggered after an existing user has been updated. Event notify about password change.

Callback Signature:
<pre><code>function($userLogin, $passwordHasBeenUpdated, $email, $password, $alias)</code></pre>

- `string` `$userLogin` The user's login handle.

- `boolean` `$passwordHasBeenUpdated` Flag containing information about password change.

## View

- [View.ReportsByDimension.render](#viewreportsbydimensionrender)

### View.ReportsByDimension.render
_Defined in [Piwik/View/ReportsByDimension](https://github.com/piwik/piwik/blob/master/core/View/ReportsByDimension.php) in line [99](https://github.com/piwik/piwik/blob/master/core/View/ReportsByDimension.php#L99)_

Triggered before rendering ReportsByDimension views. Plugins can use this event to configure ReportsByDimension instances by
adding or removing reports to display.

Callback Signature:
<pre><code>function($this)</code></pre>

- `ReportsByDimension` `$this` The view instance.

## ViewDataTable

- [ViewDataTable.addViewDataTable](#viewdatatableaddviewdatatable)
- [ViewDataTable.configure](#viewdatatableconfigure)

### ViewDataTable.addViewDataTable
_Defined in [Piwik/ViewDataTable/Manager](https://github.com/piwik/piwik/blob/master/core/ViewDataTable/Manager.php) in line [88](https://github.com/piwik/piwik/blob/master/core/ViewDataTable/Manager.php#L88)_

Triggered when gathering all available DataTable visualizations. Plugins that want to expose new DataTable visualizations should subscribe to
this event and add visualization class names to the incoming array.

**Example**

    public function addViewDataTable(&$visualizations)
    {
        $visualizations[] = 'Piwik\\Plugins\\MyPlugin\\MyVisualization';
    }

Callback Signature:
<pre><code>function(&amp;$visualizations)</code></pre>

- `array` `&$visualizations` The array of all available visualizations.

Usages:

[CoreVisualizations::getAvailableDataTableVisualizations](https://github.com/piwik/piwik/blob/master/plugins/CoreVisualizations/CoreVisualizations.php#L44)


### ViewDataTable.configure
_Defined in [Piwik/Plugin/ViewDataTable](https://github.com/piwik/piwik/blob/master/core/Plugin/ViewDataTable.php) in line [251](https://github.com/piwik/piwik/blob/master/core/Plugin/ViewDataTable.php#L251)_

Triggered during [ViewDataTable](/api-reference/Piwik/Plugin/ViewDataTable) construction. Subscribers should customize
the view based on the report that is being displayed.

Plugins that define their own reports must subscribe to this event in order to
specify how the Piwik UI should display the report.

**Example**

    // event handler
    public function configureViewDataTable(ViewDataTable $view)
    {
        switch ($view->requestConfig->apiMethodToRequestDataTable) {
            case 'VisitTime.getVisitInformationPerServerTime':
                $view->config->enable_sort = true;
                $view->requestConfig->filter_limit = 10;
                break;
        }
    }

Callback Signature:
<pre><code>function($this)</code></pre>

- `[ViewDataTable](/api-reference/Piwik/Plugin/ViewDataTable)` `$view` The instance to configure.

Usages:

[Actions::configureViewDataTable](https://github.com/piwik/piwik/blob/master/plugins/Actions/Actions.php#L138)

## WidgetsList

- [WidgetsList.addWidgets](#widgetslistaddwidgets)

### WidgetsList.addWidgets
_Defined in [Piwik/WidgetsList](https://github.com/piwik/piwik/blob/master/core/WidgetsList.php) in line [101](https://github.com/piwik/piwik/blob/master/core/WidgetsList.php#L101)_



