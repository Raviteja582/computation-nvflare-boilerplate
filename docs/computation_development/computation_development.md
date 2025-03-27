# Basic Computation Development Document

**Quick Links:**

- **[Hello World Tutorial](./tutorial_hello_world.md)** – Recommended first step.
- **[NVFLARE Programming Guide](https://nvflare.readthedocs.io/en/2.4.0/programming_guide.html)**
- **[Core Components and Workflow](./core_components_and_workflow.md)**
- **[Neuroflame Computation Interface Documentation](../neuroflame_computation_interface/neuroflame_computation_interface.md)**
- **[Development Environments](./development_environments.md)**

---

This document provides essential guidance for developing an NVFLARE app that functions as a Neuroflame computation module. It explains where to place your code and configuration files, details key programming concepts and control flow, and specifies the required methods. Use this document as your “cheat sheet” to build computation modules that run identically on the NVFLARE simulator and on the Neuroflame production platform.

---

## 1. Directory Structure

Focus on these directories when developing your computation:

- **`./app/code/`**  
  Contains your custom application code. This directory is included in the PYTHONPATH when running your NVFLARE app.

- **`./app/config/`**  
  Stores configuration files for both server and client components.

- **`./test_data/<sites>/`**  
  Holds test data for different sites. Ensure the folder names (e.g., `site1`, `site2`, `site3`) match those used with the NVFLARE command:  
  `nvflare simulator -c site1,site2,site3 ./app`

- **`./test_data/server/parameters.json`**  
  Contains the configuration parameters to be loaded into the federated learning (FL) context. This file appears in raw form on the Neuroflame desktop app, enabling the consortium leader to configure a study.

---

## 2. Programming Overview

### Key Concepts

- **Unified Operation:**  
  Your NVFLARE app is designed to run identically in both the simulator and production. The only environmental difference is in file path handling.

- **File Path Abstraction:**  
  Always use the utility functions in `./app/code/_utils/utils.py` to retrieve file paths:
  - `get_data_directory_path()`
  - `get_output_directory_path()`
  - `get_parameters_file_path()`

  These functions ensure that your app behaves consistently across environments.

---

## 3. Required Methods

Implement the following methods in your computation classes:

### Controller
- `start_controller(self, fl_ctx: FLContext) -> None`
- `stop_controller(self, fl_ctx: FLContext) -> None`
- `process_result_of_unknown_task(self, task: Task, fl_ctx: FLContext) -> None`
- `control_flow(self, abort_signal: Signal, fl_ctx: FLContext) -> None`

### Executor
- `execute(self, task_name: str, shareable: Shareable, fl_ctx: FLContext, abort_signal: Signal) -> Shareable`

### Aggregator
- `accept(self, site_result: Shareable, fl_ctx: FLContext) -> bool`
- `aggregate(self, fl_ctx: FLContext) -> Shareable`

---

## 4. Example Control Flow

A typical computation might proceed as follows:

1. **Load Configuration:**  
   Load `parameters.json` into the FL context (`fl_ctx`) so that configuration data is shared across all sites.

2. **Create and Broadcast Task:**  
   - Create a task (e.g., `TASK_NAME_GET_LOCAL_AVERAGE_AND_COUNT`).
   - Attach a callback to process site-specific results (e.g., `self._accept_site_regression_result`).
   - Broadcast the task and wait for responses using a method like `self.broadcast_and_wait`.

3. **Aggregate Results:**  
   Combine site results with `self.aggregator.aggregate()`.

4. **Distribute Global Result:**  
   - Create a second task (e.g., `TASK_NAME_ACCEPT_GLOBAL_AVERAGE`).
   - Attach the aggregated result as a shareable.
   - Broadcast this global task to all sites.

5. **Finalize Computation:**  
   End the computation after processing and confirming the global result.

---

## 5. Test Data

- **Site Data:**  
  Place test data for each site under **`./test_data/<site_name>/`**. The site folder names (e.g., `site1`, `site2`) must match those used with the NVFLARE simulator command:  
  `nvflare simulator -c site1,site2 ./app`

- **Configuration Data:**  
  Store the test version of **`parameters.json`** in **`./test_data/server/parameters.json`**. This file simulates the configuration that the controller loads into the FL context.

---

By following this document and using the provided utility functions, you can focus on developing your computation logic with confidence—knowing that your NVFLARE app will operate seamlessly in both simulation and production environments.