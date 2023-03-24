KCIDB-Grafana
=============
KCIDB-Grafana is a collection of documentation/scripts/data files needed to
setup and maintain an instance of [Grafana](https://grafana.com/) displaying
[Linux Kernel CI reports](https://staging.kernelci.org:3000/) stored in a
[KCIDB](https://github.com/kernelci/kcidb/) database.

Write to [kernelci@lists.linux.dev](mailto:kernelci@lists.linux.dev) if you want to start
submitting results from your CI system, or if you want to receive automatic
notifications of arriving results.

Architecture
------------

The current setup on [staging.kernelci.org](https://staging.kernelci.org:3000)
is implemented with a tweaked, but stock Grafana container, exposing port 3000
on localhost, and a Nginx configuration forwarding HTTPS connections on
outside port 3000 to it.

Grafana is configured with the following.

* BigQuery data source using a Google JWT file for authenticating to the KCIDB
  dataset with US as the processing location.
* A number of dashboards displaying various objects in the database, with one
  of them ("Home") set as the home dashboard.
* Anonymous access enabled.

Setup
-----
To create and start a Grafana container run:

    docker-compose up -d

For development setup, remember to adjust or drop the GF_SERVER_DOMAIN
and GF_SERVER_HTTP_PORT environment variables in `docker-compose.yml`.

Login as an administrator (default credentials are `admin`/`admin`).

### Data Source

Add BigQuery datasource, following the steps below.

1. Click "⚙️"->"Data Sources".

   ![](add_data_source_start.png)

2. Click the "Add data source" button.

   ![](add_data_source_click_button.png)

3. Search for and select the "BigQuery" data source type.

   ![](add_data_source_select_bigquery.png)

4. Set the name of the data source to "Google BigQuery" (the default), and
   optionally make it the default datasource.

   ![](add_data_source_set_name_and_make_default.png)

5. Upload Google Cloud Service Account key file. Grafana account will need
   "BigQuery Data Viewer" and "BigQuery Job User" roles only.

   ![](add_data_source_upload_key_file.png)

6. Select processing location appropriate for your dataset (US for Kernel CI
   data currently).

   ![](add_data_source_set_processing_location.png)

7. Click the "Save & Test" button.

   ![](add_data_source_save_and_test.png)

8. Verify the test was a success.

   ![](add_data_source_check_success.png)

### Dashboards

Import each of the dashboards from this repository, following the steps below.

1. Click "+"->"Import".

   ![](import_dashboard_start.png)

2. Click "Upload .json file", select dashboard file.

   ![](import_dashboard_upload_json.png)

3. Do not change properties, unless you know what you're doing.

   ![](import_dashboard_set_properties.png)

Development
-----------
To update dashboard files in this repository, export and overwrite each of
them, following the steps below.

1. Click "Share dashboard".

   ![](export_dashboard_start.png)

2. Click "Export", and then "Save to file".

   ![](export_dashboard_save_to_file.png)

### Guidelines

Each dashboard allows filtering the displayed data by time. However, only
top-level displayed objects should be filtered by that time.

E.g. if a dashboard has a list of builds then only builds started at the
specified time should appear there. However, the builds should have **all**
tests considered for the test result summary, regardless of when they started.

Yet, if the same dashboard has a panel with a list of tests across all the
builds displayed on the adjacent panel, they **should** be filtered by time,
since in that panel they would be top-level objects.

Objects without timestamp information, such as tests or builds without
`start_time`, should be displayed regardless of the selected time range, but
should be sorted after the objects with time specified, and thus more likely to
be discarded by the query's `LIMIT`.

Still, objects without timestamps should be omitted when time is a dimension
in a table or a graph.

Most dashboards allow filtering data by the origin. Again, only the
dashboard's top-level displayed objects should be filtered by origin. I.e. for
panel with a list of builds, builds should be filtered, but the tests used to
summarize test status for each of them should not, and so on, similar to time
filtering.

When a panel is implemented both on a parent object's dashboard and on a child
object's dashboard, the parent's version is considered authoritative, and
whenever both need to be updated, the parent's panel is updated, then copied
to the child, and then modified to the child's requirements (e.g. additional
filters are added, unnecessary fields are removed). This way it is easier to
keep them synced in the absence of panel/query reuse.

The dashboard filter variables (e.g. `origin`, `build_architecture`, and so
on) should be kept and passed via links between dashboards, so the user
filtering setup is preserved. Filter variables which don't make sence to some
dashboards (such as `build_architecture` for test dashboard) should be added
hidden and preserved in back links.
