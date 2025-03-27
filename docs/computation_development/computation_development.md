### Directory Structure

As a computation developer, focus on the following directories:

- **`./app/code/`**: Custom application code. This directory is included in the PYTHONPATH for NVFLARE when running your app.
- **`./app/config/`**: Contains configuration files for both server and client components.
- **`./test_data/<sites>/`**: Stores test data for different sites.
- **`./test_data/server/parameters.json`**: Parameters to be loaded into the FL context.

---

### Programming Overview

To develop a computation, review the key components and control flow. The [NVFLARE Programming Guide](https://nvflare.readthedocs.io/en/2.4.0/programming_guide.html) offers detailed instructions.

Start by reviewing: [Core Components and Workflow](./core_components_and_workflow.md).

---

### Required Methods

Implement these methods for the following classes:

#### Controller
- `start_controller(self, fl_ctx: FLContext) -> None`
- `stop_controller(self, fl_ctx: FLContext) -> None`
- `process_result_of_unknown_task(self, task: Task, fl_ctx: FLContext) -> None`
- `control_flow(self, abort_signal: Signal, fl_ctx: FLContext) -> None`

#### Executor
- `execute(self, task_name: str, shareable: Shareable, fl_ctx: FLContext, abort_signal: Signal) -> Shareable`

#### Aggregator
- `accept(self, site_result: Shareable, fl_ctx: FLContext) -> bool`
- `aggregate(self, fl_ctx: FLContext) -> Shareable`

---

### Example Control Flow

1. Load `parameters.json` into `fl_ctx` to share configuration data across sites.
2. Create a task: `TASK_NAME_GET_LOCAL_AVERAGE_AND_COUNT`.
3. Attach a callback to handle site results: `self._accept_site_regression_result`.
4. Broadcast the task and wait for responses via `self.broadcast_and_wait`.
5. Aggregate the results with `self.aggregator.aggregate()`.
6. Create a second task: `TASK_NAME_ACCEPT_GLOBAL_AVERAGE`.
7. Attach the aggregated result as a shareable and broadcast it.
8. End the computation after processing the global result.

---

### Test Data

- Place test data for each site in **`test_data/<site1, site2, etc>/`**.
- Store **`parameters.json`** in **`test_data/server/parameters.json`** for the controller to load into the shared FL context.

---

### References

- Get started with this quick tutorial: [Hello World](./tutorial_hello_world.md).
- For setup assistance: [Development Environments](./development_environments.md).

---
