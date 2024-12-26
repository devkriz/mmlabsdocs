.. Documentation for mmlabs API documentation master file, created by
   sphinx-quickstart on Thu Dec 26 21:17:49 2024.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Documentation for MotionMiners API by Process Living Lab
==========================================

Measurement Processing
--------------------

The ``measurement_processing`` module provides functions for loading and processing data from downloaded measurement files. It handles various types of time series data including activities, regions, handling heights, and more.

Key Features
~~~~~~~~~~~

- Load and process HDF5 measurement files
- Handle various time series data types (activities, regions, steps, etc.)
- Support for pause detection and removal
- Process beacon and fork height data

Core Functions
~~~~~~~~~~~~

Time Series Loading
^^^^^^^^^^^^^^^^^

.. py:function:: get_available_ts_for_measurement(measurement_file_path: str) -> list[str]

   Get the available types of timeseries data for a specified measurement file.

   :param measurement_file_path: The file path for the measurement to inspect
   :return: A list of strings with available timeseries for the measurement

.. py:function:: load_timeseries(measurement_file_path: str, target_ts_name: str, remove_pauses: bool = True) -> np.ndarray

   Load a time series from the provided HDF5 file.

   :param measurement_file_path: The fully qualified file path
   :param target_ts_name: The name of the time series file to be loaded
   :param remove_pauses: Whether to remove pause durations from the result
   :return: The requested time series data as numpy array

Activity Data
^^^^^^^^^^^

.. py:function:: get_base_activity_ts_for_measurement(measurement_file_path: str, remove_pauses: bool = True) -> np.ndarray

   Returns the activity time series for the given measurement.

   :param measurement_file_path: Path to the measurement file
   :param remove_pauses: Whether to remove pauses from the series
   :return: Activity time series in form of [<activity_id>]

Region Data
^^^^^^^^^^

.. py:function:: get_region_ts_for_measurement(measurement_file_path: str, remove_pauses: bool = True) -> np.ndarray[str]

   Returns the region time series for the given measurement.

   :param measurement_file_path: Path to the measurement file
   :param remove_pauses: Whether to remove pauses from the series
   :return: Region time series in form of [<region_uuid>]

Step and Movement Data
^^^^^^^^^^^^^^^^^^^

.. py:function:: get_step_ts_for_measurement(measurement_file_path: str, remove_pauses: bool = True) -> np.ndarray

   Returns the step time series where 1 indicates a step.

   :param measurement_file_path: Path to the measurement file
   :param remove_pauses: Whether to remove pauses from the series
   :return: Step time series in form of [0, 0, 0, 1, 0, 0, 1, 0, ...]

.. py:function:: get_walking_speed_ts_for_measurement(measurement_file_path: str, remove_pauses: bool = True) -> np.ndarray

   Returns the walking speed time series in m/s.

   :param measurement_file_path: Path to the measurement file
   :param remove_pauses: Whether to remove pauses from the series
   :return: Walking speed time series in m/s

Beacon and Fork Height Data
^^^^^^^^^^^^^^^^^^^^^^^^

.. py:function:: get_dynamic_beacon_data_for_measurement(measurement_file_path: str, remove_pauses: bool = True) -> tuple

   Returns beacon type, closeness and usage info for all dynamic beacons.

   :param measurement_file_path: Path to the measurement file
   :param remove_pauses: Whether to remove pauses from the series
   :return: Tuple of (closeness_arr, usage_arr, uuids)

.. py:function:: get_fork_height_data_for_measurement(measurement_file_path: str, remove_pauses: bool = True) -> tuple

   Returns fork height level data for fork lift vehicles.

   :param measurement_file_path: Path to the measurement file
   :param remove_pauses: Whether to remove pauses from the series
   :return: Tuple of (fork_lvl_arr, fork_target_lvl_arr, beacon_uuids, beacon_type_ids, target_heights)

Data Management
-------------

The ``data`` module provides the ``MMLabsData`` class for managing measurement data, process metadata, and layout images.

.. py:class:: MMLabsData(api=None, data_dir="data/", offline_mode=False, verbose=False)

   Main class for handling MMLabs measurement data and process metadata.

   :param api: Optional MMLabsAPI instance
   :param data_dir: Directory for caching downloaded data
   :param offline_mode: If True, operates without API connections
   :param verbose: Enable verbose logging

   .. py:method:: add_api_token(token: str)

      Add a new token for process data access. Retrieves and caches process metadata.

      :param token: Base64-encoded MPI access token
      :raises MMLabsException: If token is invalid or process metadata cannot be retrieved

   .. py:method:: get_layout_image(process_uuid: str) -> bytes

      Get the layout image associated with a process.

      :param process_uuid: Process UUID
      :return: Raw image bytes
      :raises MMLabsException: If process UUID is invalid or image cannot be retrieved

   .. py:method:: get_measurements_df(process_uuid: str) -> pandas.DataFrame

      Get measurements as a sorted pandas DataFrame.

      :param process_uuid: Process UUID
      :return: DataFrame with measurement data, sorted by timestamp
      :raises MMLabsException: If process UUID is not set

   .. py:method:: get_api_all_data(process_uuid: str, use_cache: bool = True)

      Download all process data including metadata, measurements, and layout images.

      :param process_uuid: Process UUID to retrieve data for
      :param use_cache: If True, skip downloading files that exist locally
      :raises MMLabsException: If offline mode is enabled

   .. py:method:: get_measurement_dir_path(process_uuid: str, measurement_uuid: str) -> str

      Get the filesystem path for a specific measurement.

      :param process_uuid: Process UUID
      :param measurement_uuid: Measurement UUID
      :return: Full path to measurement directory

Data Structure
~~~~~~~~~~~~

The MMLabsData class maintains several key data structures:

- **processes**: Dictionary mapping process UUIDs to process metadata
- **measurements**: Dictionary mapping process UUIDs to measurement data
- **tokens**: Dictionary storing API tokens for each process

Example Usage
~~~~~~~~~~~

.. code-block:: python

   from mm_labs.data import MMLabsData

   # Initialize data manager
   data_manager = MMLabsData(data_dir="./cache")

   # Add API token for access
   data_manager.add_api_token("your-api-token")

   # Download all data for a process
   data_manager.get_api_all_data("process-uuid")

   # Get measurements as DataFrame
   measurements_df = data_manager.get_measurements_df("process-uuid")

   # Get layout image
   layout_bytes = data_manager.get_layout_image("process-uuid")

Cache Structure
~~~~~~~~~~~~~

The cache directory structure is organized as follows::

    data_dir/
    ├── <process_uuid>/
    │   ├── process_metadata.json
    │   ├── measurements.json
    │   ├── token.json
    │   ├── layout.png
    │   └── <measurement_uuid>/
    │       ├── activity_data
    │       ├── region_data
    │       └── other measurement files...

API Interface
-----------

The ``api`` module provides the ``MMLabsAPI`` class for interacting with the MMLabs REST API.

.. py:class:: MMLabsAPI(base_url='https://mpi.motionminers.com/api/4labs/v2/', token=None)

   Main class for making API calls to the MMLabs server.

   :param base_url: The MMLabs API endpoint URL
   :param token: Optional Base64-encoded authorization token

   .. py:method:: set_token(token: str)

      Set the authorization token for API interactions.

      :param token: Base64-encoded authorization token

   .. py:method:: get_layout_image() -> bytes

      Retrieve the layout image from the API.

      :return: Image as byte array
      :raises MMLabsException: If token is not set or API call fails

   .. py:method:: get_measurements() -> list

      Retrieve list of available measurements from the API.

      :return: List of measurement objects
      :raises MMLabsException: If token is not set or API call fails

   .. py:method:: get_measurement(uuid: str, filename: str) -> bytes

      Download a specific measurement file from the API.

      :param uuid: Measurement UUID
      :param filename: Name of the file to download
      :return: File contents as byte array
      :raises MMLabsException: If UUID/filename is missing or API call fails

   .. py:method:: get_process_metadata() -> dict

      Retrieve process metadata from the API.

      :return: Process metadata dictionary
      :raises MMLabsException: If token is not set or API call fails

API Endpoints
~~~~~~~~~~~

The MMLabsAPI class interacts with the following endpoints:

- ``GET /layout_image``: Retrieve layout image
- ``GET /measurements``: List available measurements
- ``GET /measurements/<uuid>/files/<filename>``: Download measurement file
- ``GET /process_metadata``: Get process metadata

Authentication
~~~~~~~~~~~~

All API calls require authentication using a Bearer token:

.. code-block:: python

   api = MMLabsAPI()
   api.set_token("your-base64-encoded-token")

Example Usage
~~~~~~~~~~~

.. code-block:: python

   from mm_labs.api import MMLabsAPI

   # Initialize API client
   api = MMLabsAPI()
   api.set_token("your-token")

   # Get process metadata
   metadata = api.get_process_metadata()

   # List measurements
   measurements = api.get_measurements()

   # Download measurement file
   file_data = api.get_measurement(
       uuid="measurement-uuid",
       filename="activity_data"
   )

   # Get layout image
   image_bytes = api.get_layout_image()

Error Handling
~~~~~~~~~~~~

The API client raises ``MMLabsException`` in the following cases:

- Token not set
- Missing required parameters
- API request failures
- Invalid responses

Example error handling:

.. code-block:: python

   from mm_labs.exceptions import MMLabsException

   try:
       api.get_process_metadata()
   except MMLabsException as e:
       print(f"API error: {e}")

Map Visualization
--------------

The ``map`` module provides the ``MMLabsMap`` class for visualizing layout data, beacons, and regions using interactive maps.

.. py:class:: MMLabsMap(data: MMLabsData, process_uuid: str)

   Main class for creating interactive maps with layout visualization.

   :param data: MMLabsData instance containing process data
   :param process_uuid: UUID of the process to visualize

   .. py:method:: create_map()

      Creates the base map with layout image overlays.

   .. py:method:: create_beacon_layer()

      Creates a layer showing beacons on the map with interactive markers.

   .. py:method:: create_region_layer()

      Creates a layer showing regions as interactive polygons.

   .. py:method:: set_heatmap(data: dict = None)

      Sets up a heatmap visualization for regions.

      :param data: Dictionary mapping region IDs to values (0.0 to 1.0)

   .. py:method:: get_map() -> ipyleaflet.Map

      Returns the interactive map object.

      :return: ipyleaflet Map instance

Features
~~~~~~~~

- Interactive map visualization using ipyleaflet
- Layout image overlay with proper georeferencing
- Beacon positioning with interactive markers
- Region visualization with hover effects
- Heatmap support for region data
- Information popup for regions and beacons

Example Usage
~~~~~~~~~~~

.. code-block:: python

   from mm_labs.map import MMLabsMap
   from mm_labs.data import MMLabsData

   # Initialize data manager
   data_manager = MMLabsData(data_dir="./cache")
   data_manager.add_api_token("your-api-token")

   # Create map for a process
   process_map = MMLabsMap(data_manager, "process-uuid")

   # Add heatmap data
   heatmap_data = {
       "1": 0.8,
       "2": 0.5,
       "3": 0.2
   }
   process_map.set_heatmap(heatmap_data)

   # Display the map (in Jupyter notebook)
   process_map.get_map()

Map Components
~~~~~~~~~~~~

The map visualization includes several layers:

- **Base Map**: OpenStreetMap background
- **Layout**: Process layout image overlay
- **Beacons**: Interactive markers showing beacon positions
- **Regions**: Interactive polygons showing defined regions
- **Heatmap**: Optional choropleth visualization of region data

Dependencies
~~~~~~~~~~

The map module requires:

- ipyleaflet
- ipywidgets
- branca
- PIL (Python Imaging Library)
- numpy

Exception Handling
---------------

The ``exceptions`` module provides a set of custom exceptions for handling various error cases in the MMLabs API.

Base Exceptions
~~~~~~~~~~~~

.. py:exception:: MMLabsException(message: str)

   Base exception class for all MMLabs-specific errors.

   :param message: Error message to display

Click Exceptions
~~~~~~~~~~~~~

.. py:exception:: ClickException(message: str)

   Base exception that Click can handle and show to the user.

   :param message: Error message to display
   :ivar exit_code: The exit code for this exception (default: 1)

   .. py:method:: format_message() -> str

      Returns the formatted error message.

   .. py:method:: show(file: Optional[IO] = None) -> None

      Display the error message to the user.

      :param file: File-like object to write to (defaults to stderr)

.. py:exception:: UsageError(message: str, ctx: Optional[Context] = None)

   Signals a usage error, typically aborting further handling.

   :param message: Error message to display
   :param ctx: Optional Click context that caused the error
   :ivar exit_code: The exit code for this exception (default: 2)

.. py:exception:: BadParameter(message: str, ctx: Optional[Context] = None, param: Optional[Parameter] = None, param_hint: Optional[str] = None)

   Formats a standardized error message for invalid parameter values.

   :param message: Error message to display
   :param ctx: Optional Click context
   :param param: Parameter object that caused the error
   :param param_hint: Optional string to show as parameter name

.. py:exception:: MissingParameter(message: Optional[str] = None, ctx: Optional[Context] = None, param: Optional[Parameter] = None, param_hint: Optional[str] = None, param_type: Optional[str] = None)

   Raised when a required option or argument is not provided.

   :param message: Error message to display
   :param ctx: Optional Click context
   :param param: Parameter object that's missing
   :param param_hint: Optional string to show as parameter name
   :param param_type: Type of parameter ("parameter", "option", or "argument")

File Handling Exceptions
~~~~~~~~~~~~~~~~~~~~

.. py:exception:: FileError(filename: str, hint: Optional[str] = None)

   Raised when a file cannot be opened.

   :param filename: Name of the file that couldn't be opened
   :param hint: Optional hint about the error (defaults to "unknown error")

Example Usage
~~~~~~~~~~~

.. code-block:: python

   try:
       # Attempt API operation
       api.get_process_metadata()
   except MMLabsException as e:
       print(f"API error: {e}")
   except FileError as e:
       print(f"File error: {e.format_message()}")
   except UsageError as e:
       e.show()  # Shows formatted error with usage hints

Error Messages
~~~~~~~~~~~~

Examples of formatted error messages:

- Missing Parameter::

    Error: Missing option "--token".

- Bad Parameter::

    Error: Invalid value for "--process-id": Process UUID not found.

- File Error::

    Error: Could not open file 'data.json': Permission denied.

Translations
----------

The ``translations`` module provides mappings between internal activity keys and their human-readable names.

Constants
~~~~~~~~

.. py:data:: ACTIVITY_TRANSLATION_KEY_TO_NAME

   Dictionary mapping internal activity keys to human-readable names.

   .. code-block:: python

      ACTIVITY_TRANSLATION_KEY_TO_NAME = {
          'HUMAN_ACT.WALK': 'Walk',
          'HUMAN_ACT.OTHER': 'Unknown',
          'HUMAN_ACT.STAND': 'Stand',
          'HUMAN_ACT.HANDLE': 'Handling',
          'HUMAN_ACT.DRIVE': 'Drive',
          'HUMAN_ACT.NO_HANDLE': 'No Handling',
          'HUMAN_ACT.HANDLE_UP': 'Handle up',
          'HUMAN_ACT.HANDLE_CENTER': 'Handle center',
          'HUMAN_ACT.HANDLE_DOWN': 'Handle down'
      }

Example Usage
~~~~~~~~~~~

.. code-block:: python

   from mm_labs.translations import ACTIVITY_TRANSLATION_KEY_TO_NAME

   # Convert activity key to human-readable name
   activity_key = 'HUMAN_ACT.WALK'
   activity_name = ACTIVITY_TRANSLATION_KEY_TO_NAME[activity_key]
   print(f"Activity: {activity_name}")  # Output: Activity: Walk

   # Get all available activities
   available_activities = ACTIVITY_TRANSLATION_KEY_TO_NAME.values()

User Interface
------------

The ``ui`` module provides the ``MMLabsUI`` class for creating interactive Jupyter notebook interfaces for MMLabs data visualization and management.

.. py:class:: MMLabsUI(data=None)

   Main class for creating interactive UI components in Jupyter notebooks.

   :param data: Optional MMLabsData instance for data management

   .. py:method:: show_configurator() -> widgets.VBox

      Creates and returns a configuration widget with token management and data selection.

      :return: IPython widget containing the configuration interface

   .. py:method:: show_map() -> ipyleaflet.Map

      Creates and returns an interactive map visualization for the selected process.

      :return: Interactive map widget
      :raises MMLabsException: If no process is selected

   .. py:method:: show_configurator_sidecar(anchor: str = "right")

      Displays the configuration widget in a sidecar panel.

      :param anchor: Position of the sidecar ("right" or "left")

   .. py:method:: show_map_sidecar(anchor: str = "right", listen_to_selection: bool = True)

      Displays the map widget in a sidecar panel.

      :param anchor: Position of the sidecar ("right" or "left")
      :param listen_to_selection: Whether to update on process selection changes

   .. py:method:: show_image(image_data: bytes) -> PIL.Image

      Displays image data as a PIL Image.

      :param image_data: Raw image bytes
      :return: PIL Image object

   .. py:method:: show_json_beautified(json_data: dict) -> widgets.JSON

      Displays JSON data in a formatted widget.

      :param json_data: Dictionary to display
      :return: JSON display widget

Features
~~~~~~~~

- Interactive process and measurement selection
- Token management interface
- Map visualization in sidecar panels
- Real-time updates on selection changes
- Image and JSON data visualization
- Offline mode support

Example Usage
~~~~~~~~~~~

.. code-block:: python

   from mm_labs.ui import MMLabsUI
   from mm_labs.data import MMLabsData

   # Initialize data manager
   data_manager = MMLabsData(data_dir="./cache")
   data_manager.add_api_token("your-api-token")

   # Create UI
   ui = MMLabsUI(data_manager)

   # Show configuration panel
   ui.show_configurator_sidecar()

   # Show map visualization
   ui.show_map_sidecar()

UI Components
~~~~~~~~~~~

The UI consists of several key components:

- **Token Management**:
   - Token input field
   - Add token button
   - Process metadata download

- **Data Selection**:
   - Process dropdown
   - Measurement dropdown
   - Offline mode toggle
   - Refresh button
   - Download button

- **Map Visualization**:
   - Interactive map display
   - Layout overlay
   - Beacon markers
   - Region highlighting

Dependencies
~~~~~~~~~~

The UI module requires:

- ipywidgets
- sidecar
- IPython
- PIL (Python Imaging Library)
- pandas

.. toctree::
   :maxdepth: 2
   :caption: Contents:

