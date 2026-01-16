# Evaluating LLM Agents for AIOps on Red Hat OpenShift

This repository is the `Operationalizing AIOps with LLM Agents and Predictive Models: A Case Study in Kubernetes Environments` paper experiments verification repository.

_Note: The paper is not yet published, however it has been sent for review at the [FSE2026 conference](https://conf.researchr.org/track/fse-2026/fse-2026-industry-papers), industry track. If accepted, as soon as that process completes, publication details shall be added here._

In order to execute the experiments, access to a Red Hat OpenShift environment is required. If you don't have one, you can start a trial by creating an account at [RedHat Cloud website](https://cloud.redhat.com). Alternatively, it is also possible to execute the guide on a local (reduced version) of the platform, typically used for development purposes. Once you registered on the [RedHat Cloud website](https://cloud.redhat.com), you can obtain your version of the local Red Hat Openshift by downloading the installer from the [Console](https://console.redhat.com/openshift/create/local).

## Foreword

This repository contains the following:

- The research questions related detailed information that due to space limitations was not included in the paper:
- The experiment code (Jupyter notebook) and the results (data) used to fill out the information in the paper and in this README.
- Additional supporting code (for the tools used to carry out the experiments).
- A quick guide for how to configure a Red Hat Openshift deployment to:
  - Set up an environment for creating and collecting data required by the MLASP tool.
  - Set up an environment to execute the LLM experiments for the provided notebooks.

## Tool list for LLM agents experiments

This section provides the list of tools used in the experiments alongside a brief description of the respective tool. The source code of each tool is availble in the notebooks section. We number the tools as $T<n>$, with $n \in [1..9]$. We use this numbering system in the next section to describe the relationship between the experiment queries and the list of tools.

1. **T1, [MLASP](https://petertsehsun.github.io/papers/MLASP_EMSE2021.pdf):** is a machine learning based capacity planning tool that generates a set of parameter configurations (e.g., size of a thread pool) to support a desired KPI value for a generic WireMock application. A complete implementation for OpenShift (including data generation and model creation) can be found [here](https://github.com/eartvit/mlasp-on-ocp).

2. **T2, RAG:** a tool that gives the LLM the ability to search a specialized vector database that contains encoded documentation about the Red Hat OpenShift AI operator, including procedure description and how-to information. The LLM can inspect this database to obtain information based on the received query and then summarize a response to the user. A complete implementation of this tool, for the purposes of the experiments conducted for this paper can be found [here](https://github.com/eartvit/llm-on-ocp). The notebook that performs the data ingestion is [here](https://github.com/eartvit/llm-on-ocp/blob/main/notebooks/Milvus-ingest-LangChain.ipynb), however you must change the following variable: `product_version= 2.8` for the experiments to work.

3. **T3, Time info:** a tool that calculates the timestamp, the iso-formatted string, and the timezone string of the requested time information. Returns a Python object containing the timestamp value, the ISO formatted string of the date-time value, and the timezone string.

4. **T4, List operators:** is used to find information about operators installed within a namespace. The response may contain information such as the name of the operator, its version, and deployment status.

5. **T5, Pod summary:** is used to summarize information about the pods that exist in a namespace. A pod is a Kubernetes abstraction that represents a group of one or more application containers and some shared resources for those containers (e.g., shared storage, networking cluster, etc.). The tool returns an object containing the name of the namespace and pod state (e.g., running, stopped) and pod counter information. For the running pods, it also returns its name and, if available, any service information such as service name, service ports, and route.

6. **T6, Service summary:** is used to summarize service information in an OpenShift namespace. It returns an object containing the name of the namespace and a list of the available services and their properties, such as name, port numbers, and route information.

7. **T7, Prometheus metric names:** list available metric names in a Prometheus~\cite{prometheus} instance using an input filter. Prometheus is an open-source technology designed to provide monitoring and alerting functionality for cloud-native environments, including Kubernetes. The inputs for this tools are a filter name and value for the filter (e.g. input filter name is 'namespace' and filter value is 'demo'). It returns a list containing the available metric names.

8. **T8, Prometheus metric data range:** is used to list the application metric values and associated timestamps between a start and an end timestamp interval for a given metric name stored within a Prometheus instance. It returns a pydantic object containing the list of the desired application metric values and associated timestamp information.

9. **T9, Plot prometheus metric range data as file:** is used to create a file with the plot of the instantaneous rate (irate) of an application metric values and associated timestamps between a start and an end timestamp interval for a given metric name stored within a Prometheus instance. It returns a string containing the name of the file containing the plot.

## Experiments detailed results

This section provides the detailed results of different LLMs performance over each of the 25 queries comprising the experiments. To evaluate each model's robustness and consistency, we ask each model the same question 10 times. We evaluate our queries on the following list of commonly-used LLMs: from the Anthropic family: Claude 3.5 Sonnet, Claude 3 Haiku, and Claude 3 Opus; from the Mistral family: Mistral Largest, Mixtral 8x22B, and Mistral Small 7B; from the OpenAI family: GPT 3.5 Turbo, GPT 4-o, GPT 4-o Mini, and GPT 4 Turbo.

The queries are a range of task types—general-purpose prompts, platform-specific commands, and application-level diagnostic requests. In order to respond to these queries, the LLMs must use their training data, or the available list of tools. Additionally, we categorize these queries into two distinct types: **Simple Reasoning (SR)**: where the LLM must respond by relying solely on its training data or, at most, utilizing one tool; and **Advanced Reasoning (AR)**: where the LLM must identify multiple tools to use and construct a workflow in which the tools are employed in the correct sequence. In advanced reasoning scenarios, the LLM is also responsible for formatting the data as required before using a tool, and the same tool may be invoked multiple times with varying inputs.

Below is the list of queries and their respective category information and tool list assocation:

| Query \#. | Cat. | Tool List  | Query Text                                                                                                                                                                                                                                                                             |
| --------- | ---- | ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Q-01      | SR   | -          | Give me the list of tools you have access to.                                                                                                                                                                                                                                          |
| Q-02      | SR   | -          | Give me the list and a short description of the tools you have access to.                                                                                                                                                                                                              |
| Q-03      | SR   | T4         | What operators are in namespace demo?                                                                                                                                                                                                                                                  |
| Q-04      | SR   | T4         | What operators are in namespace demo? Please provide only the name and the version for each operator                                                                                                                                                                                   |
| Q-05      | SR   | T2         | How can I create a Data Science Project?                                                                                                                                                                                                                                               |
| Q-06      | SR   | T5         | Tell me about the pods in namespace demo.                                                                                                                                                                                                                                              |
| Q-07      | SR   | T5         | Give me a summary of the running pods in namespace demo. Please include service and route information in the response.                                                                                                                                                                 |
| Q-08      | SR   | T5         | Give me the complete summary of the pods in namespace demo.                                                                                                                                                                                                                            |
| Q-09      | SR   | T5         | Give me a summary of the running pods in namespace demo. Give me only the names and the route if they have one.                                                                                                                                                                        |
| Q-10      | SR   | T3         | What is the current date time?                                                                                                                                                                                                                                                         |
| Q-11      | SR   | T3         | What is the current timestamp?                                                                                                                                                                                                                                                         |
| Q-12      | SR   | T3         | What is the timestamp and date time for 3 hours ago?                                                                                                                                                                                                                                   |
| Q-13      | SR   | T3         | What is the timestamp and date time for 3 hours from now?                                                                                                                                                                                                                              |
| Q-14      | SR   | T3         | What is the timestamp and date time for 3 hours ago?                                                                                                                                                                                                                                   |
| Q-15      | SR   | T6         | Is there a prometheus service running in namespace demo? If so, give me its name and port values.                                                                                                                                                                                      |
| Q-16      | AR   | T6, T7     | Find out the service name and port number of the Prometheus service running in namespace demo. Then use that information to retrieve the list of metrics filtered by namespace demo.                                                                                                   |
| Q-17      | AR   | T6, T7     | Find out the Prometheus service name and port number running in namespace demo. Give me all the metrics stored by it that have a name that starts with load_generator.                                                                                                                 |
| Q-18      | SR   | T1         | What configuration of WireMock supports a throughput KPI of 307 within a 2.9 percent precision? Search for 100 epochs to find the result.                                                                                                                                              |
| Q-19      | AR   | T3, T6, T9 | Find out the Prometheus service name and port number running in namespace demo. Use it to to plot all the prometheus metric data for the metric load_generator_total_msg starting 40 days ago until now. Return only the file name and nothing else.                                   |
| Q-20      | AR   | T3, T6, T8 | Find out the Prometheus service name and port number running in namespace demo. Use that to get all the prometheus metric data for the metric load_generator_total_msg starting 40 days ago until now. Print out only the metric values and their associated timestamp as a CSV table. |

### RQ1: How accurately do different LLMs perform on a set of IT Operations tasks?

The below table presents the accuracy values associated with each model as a summary of the results groupped by type of tasks:

|       Model       |   SR Task    |  AR Task   |
| :---------------: | :----------: | :--------: |
|                   | (0..1 tools) | (2+ tools) |
| Claude 3.5 Sonnet |   100\%      |    95\%    |
|  Claude 3 Haiku   |   93.13\%    |    30\%    |
|   Claude 3 Opus   |   93.13\%    |    75\%    |
|  Mistral Largest  |   99.38\%    |    45\%    |
|   Mixtral 8x22B   |   12.50\%    |    0\%     |
| Mistral Small 7B  |   81.25\%    |    0\%     |
|   GPT 3.5 Turbo   |   87.5\%     |    0\%     |
|      GPT 4-o      |   95.63\%    |   100\%    |
|   GPT 4-o Mini    |   93.75\%    |   77.5\%   |
|    GPT 4 Turbo    |   93.75\%    |    90\%    |

The below table presents the detailed accuracy values for each query and LLM tested:

| Query \# | Claude 3.5 Sonnet | Claude 3 Haiku | Claude 3 Opus | Mistral Largest | Mixtral 8x22B | Mistral Small (7B) | GPT 3.5 Turbo | GPT 4-o | GPT 4-o Mini | GPT 4 Turbo |
| -------- | ----------------- | -------------- | ------------- | --------------- | ------------- | ------------------ | ------------- | ------- | ------------ | ----------- |
| Q-01     | 100.00%           | 100.00%        | 100.00%       | 100.00%         | 100.00%       | 100.00%            | 100.00%       | 100.00% | 100.00%      | 100.00%     |
| Q-02     | 100.00%           | 100.00%        | 100.00%       | 100.00%         | 100.00%       | 100.00%            | 0.00%         | 100.00% | 100.00%      | 100.00%     |
| Q-03     | 100.00%           | 90.00%         | 100.00%       | 100.00%         | 0.00%         | 100.00%            | 100.00%       | 100.00% | 100.00%      | 100.00%     |
| Q-04     | 100.00%           | 100.00%        | 100.00%       | 100.00%         | 0.00%         | 100.00%            | 100.00%       | 100.00% | 100.00%      | 100.00%     |
| Q-05     | 100.00%           | 0.00%          | 0.00%         | 90.00%          | 0.00%         | 0.00%              | 100.00%       | 30.00%  | 0.00%        | 0.00%       |
| Q-06     | 100.00%           | 100.00%        | 100.00%       | 100.00%         | 0.00%         | 100.00%            | 100.00%       | 100.00% | 100.00%      | 100.00%     |
| Q-07     | 100.00%           | 100.00%        | 100.00%       | 100.00%         | 0.00%         | 100.00%            | 100.00%       | 100.00% | 100.00%      | 100.00%     |
| Q-08     | 100.00%           | 100.00%        | 100.00%       | 100.00%         | 0.00%         | 100.00%            | 100.00%       | 100.00% | 100.00%      | 100.00%     |
| Q-09     | 100.00%           | 100.00%        | 100.00%       | 100.00%         | 0.00%         | 100.00%            | 100.00%       | 100.00% | 100.00%      | 100.00%     |
| Q-10     | 100.00%           | 100.00%        | 100.00%       | 100.00%         | 0.00%         | 0.00%              | 100.00%       | 100.00% | 100.00%      | 100.00%     |
| Q-11     | 100.00%           | 100.00%        | 100.00%       | 100.00%         | 0.00%         | 100.00%            | 100.00%       | 100.00% | 100.00%      | 100.00%     |
| Q-12     | 100.00%           | 100.00%        | 100.00%       | 100.00%         | 0.00%         | 100.00%            | 100.00%       | 100.00% | 100.00%      | 100.00%     |
| Q-13     | 100.00%           | 100.00%        | 100.00%       | 100.00%         | 0.00%         | 100.00%            | 100.00%       | 100.00% | 100.00%      | 100.00%     |
| Q-14     | 100.00%           | 100.00%        | 100.00%       | 100.00%         | 0.00%         | 100.00%            | 100.00%       | 100.00% | 100.00%      | 100.00%     |
| Q-15     | 100.00%           | 100.00%        | 100.00%       | 100.00%         | 0.00%         | 0.00%              | 0.00%         | 100.00% | 100.00%      | 100.00%     |
| Q-16     | 80.00%            | 10.00%         | 0.00%         | 0.00%           | 0.00%         | 0.00%              | 0.00%         | 100.00% | 100.00%      | 100.00%     |
| Q-17     | 100.00%           | 50.00%         | 100.00%       | 100.00%         | 0.00%         | 0.00%              | 0.00%         | 100.00% | 100.00%      | 100.00%     |
| Q-18     | 100.00%           | 100.00%        | 90.00%        | 100.00%         | 0.00%         | 100.00%            | 100.00%       | 100.00% | 100.00%      | 100.00%     |
| Q-19     | 100.00%           | 30.00%         | 100.00%       | 80.00%          | 0.00%         | 0.00%              | 0.00%         | 100.00% | 100.00%      | 80.00%      |
| Q-20     | 100%\*            | 30%\*          | 100%\*        | 0.00%           | 0.00%         | 0.00%              | 0.00%         | 100.00% | 10.00%       | 80.00%      |

### RQ2: How fast do different LLMs perform on a set of IT Operations tasks?

The below table presents the response times in the P50 and P90 quantiles associated with each model as a summary of the results groupped by type of tasks:
| Model | Metric (Average) | SR Task (0..1 tools) | AR Task (2+ tools) |
|---|---|---|---|
| | | | |
| Claude 3.5 Sonnet | P-50 | 7.17 | 19.14 |
| | P-90 | 7.98 | 20.54 |
| | | | |
| Claude 3 Haiku | P-50 | 3.60 | 9.07 |
| | P-90 | 4.91 | 16.08 |
| | | | |
| Claude 3 Opus | P-50 | 19.50 | 48.38 |
| | P-90 | 22.12 | 55.12 |
| | | | |
| Mistral Largest | P-50 | 8.48 | 71.95 |
| | P-90 | 12.93 | 88.04 |
| | | | |
| Mixtral 8x22B | P-50 | 5.02 | 6.88 |
| | P-90 | 6.19 | 7.17 |
| | | | |
| Mistral Small 7B | P-50 | 5.35 | 9.12 |
| | P-90 | 5.84 | 9.42 |
| | | | |
| GPT 3.5 Turbo | P-50 | 4.02 | 6.99 |
| | P-90 | 4.93 | 7.35 |
| | | | |
| GPT 4-o | P-50 | 5.26 | 46.12 |
| | P-90 | 6.71 | 54.42 |
| | | | |
| GPT 4-o Mini | P-50 | 4.92 | 22.08 |
| | P-90 | 7.14 | 28.61 |
| | | | |
| GPT 4 Turbo | P-50 | 10.58 | 48.62 |
| | P-90 | 11.96 | 54.99 |

The below table presents the detailed response time values (in seconds) for each query and LLM tested:

| Query \# | Metric | Claude 3.5 Sonnet | Claude 3 Haiku | Claude 3 Opus | Mistral Largest | Mixtral 8x22B | Mistral Small (7B) | GPT 3.5 Turbo | GPT 4-o | GPT 4-o Mini | GPT 4 Turbo |
| -------- | ------ | ----------------- | -------------- | ------------- | --------------- | ------------- | ------------------ | ------------- | ------- | ------------ | ----------- |
|          | P-50   | 3.86              | 2.79           | 20.63         | 11.1            | 5.17          | 2.72               | 2.06          | 4.77    | 4.4          | 10.96       |
| Q-01     | P-90   | 6.22              | 5.28           | 27.5          | 27.96           | 5.82          | 2.95               | 2.38          | 5.67    | 6.39         | 12.56       |
|          | Max    | 6.35              | 6.84           | 28.23         | 42.01           | 6.22          | 3.26               | 2.44          | 6       | 9.74         | 12.64       |
|          |        |                   |                |               |                 |               |                    |               |         |              |             |
|          | P-50   | 7                 | 4.13           | 24.35         | 25.17           | 7.97          | 6.21               | 1.64          | 8.39    | 5.48         | 14.84       |
| Q-02     | P-90   | 8.46              | 4.97           | 27.31         | 33.17           | 8.24          | 6.96               | 1.91          | 10.49   | 10.8         | 16.3        |
|          | Max    | 9.03              | 7.47           | 33.9          | 34.49           | 8.26          | 7.18               | 1.95          | 15.49   | 43.94        | 19.6        |
|          |        |                   |                |               |                 |               |                    |               |         |              |             |
|          | P-50   | 5.99              | 2.9            | 17.73         | 5.31            | 2.29          | 6.42               | 3.77          | 5.47    | 4.08         | 12.63       |
| Q-03     | P-90   | 7.37              | 3.88           | 20.6          | 6.62            | 2.85          | 6.89               | 4.36          | 6.43    | 6.84         | 13.62       |
|          | Max    | 7.49              | 4.13           | 20.94         | 7.53            | 3.11          | 7.08               | 4.38          | 6.67    | 10.8         | 13.62       |
|          |        |                   |                |               |                 |               |                    |               |         |              |             |
|          | P-50   | 4.95              | 2.09           | 16.37         | 5.53            | 2.53          | 4.86               | 4.02          | 4.12    | 3.76         | 9.48        |
| Q-04     | P-90   | 5.18              | 2.36           | 18.53         | 9.46            | 2.85          | 5.11               | 4.73          | 4.98    | 5.09         | 11.73       |
|          | Max    | 5.27              | 2.52           | 18.92         | 9.54            | 3.14          | 5.64               | 4.76          | 5.59    | 6.45         | 12.84       |
|          |        |                   |                |               |                 |               |                    |               |         |              |             |
|          | P-50   | 7.53              | 3.14           | 23.69         | 6.46            | 7.08          | 5.56               | 3.59          | 8.19    | 9.46         | 25.81       |
| Q-05     | P-90   | 8.62              | 4.95           | 26.87         | 11.74           | 7.99          | 6.36               | 4.08          | 11.28   | 17.28        | 28.15       |
|          | Max    | 8.79              | 5.77           | 32.01         | 11.88           | 9.1           | 6.72               | 4.58          | 13.66   | 17.55        | 29.08       |
|          |        |                   |                |               |                 |               |                    |               |         |              |             |
|          | P-50   | 10.61             | 6.36           | 22.38         | 13.58           | 6.09          | 8.53               | 8.28          | 10.31   | 8.98         | 16.17       |
| Q-06     | P-90   | 10.97             | 7.06           | 23.45         | 18.39           | 6.36          | 8.85               | 10.6          | 12.01   | 14.82        | 18.78       |
|          | Max    | 11.03             | 7.27           | 23.72         | 25.84           | 6.86          | 8.93               | 10.9          | 15.89   | 29.5         | 19.26       |
|          |        |                   |                |               |                 |               |                    |               |         |              |             |
|          | P-50   | 10.52             | 5.93           | 22.81         | 12.27           | 4.7           | 9.01               | 8.19          | 8.23    | 8.67         | 15.19       |
| Q-07     | P-90   | 11.17             | 8.42           | 23.28         | 14.58           | 4.89          | 10.02              | 8.94          | 9.36    | 9.86         | 16.32       |
|          | Max    | 11.61             | 10.04          | 23.49         | 22.14           | 5.16          | 10.3               | 9.08          | 10.58   | 10.31        | 17.42       |
|          |        |                   |                |               |                 |               |                    |               |         |              |             |
|          | P-50   | 11.04             | 6.21           | 22            | 11.3            | 3.52          | 8.64               | 7.74          | 8.71    | 8.81         | 15.78       |
| Q-08     | P-90   | 11.72             | 6.98           | 26.9          | 19.09           | 15.42         | 8.88               | 8.57          | 12.04   | 10.93        | 17.68       |
|          | Max    | 12.36             | 7.17           | 27.72         | 20.55           | 120.12        | 9.04               | 8.92          | 16.6    | 11.75        | 20.88       |
|          |        |                   |                |               |                 |               |                    |               |         |              |             |
|          | P-50   | 8.33              | 5.45           | 20.05         | 8.17            | 3.37          | 6.94               | 6.69          | 6.65    | 6.46         | 12.14       |
| Q-09     | P-90   | 9.07              | 6.89           | 22.06         | 15.11           | 3.74          | 7.54               | 7.14          | 8.09    | 8.43         | 13.01       |
|          | Max    | 9.32              | 7.27           | 22.96         | 15.48           | 4.49          | 7.74               | 7.58          | 9.36    | 11.3         | 13.76       |
|          |        |                   |                |               |                 |               |                    |               |         |              |             |
|          | P-50   | 5.39              | 1.86           | 16.8          | 3.44            | 2.53          | 2.55               | 1.77          | 2.07    | 1.7          | 3.53        |
| Q-10     | P-90   | 5.81              | 3.32           | 17.29         | 5.11            | 3.28          | 2.97               | 1.95          | 2.54    | 2.49         | 4.81        |
|          | Max    | 5.83              | 4.02           | 18.1          | 10.68           | 3.38          | 3.11               | 2.03          | 3.02    | 6.39         | 5.57        |
|          |        |                   |                |               |                 |               |                    |               |         |              |             |
|          | P-50   | 5.55              | 2.29           | 16.24         | 3.81            | 3.33          | 2.98               | 2.06          | 2.19    | 1.97         | 4.41        |
| Q-11     | P-90   | 5.94              | 3.83           | 19.09         | 4.88            | 3.51          | 3.4                | 2.33          | 3.3     | 3.43         | 5.17        |
|          | Max    | 6.1               | 5.85           | 19.31         | 5.15            | 3.69          | 3.52               | 2.5           | 4.39    | 3.62         | 5.69        |
|          |        |                   |                |               |                 |               |                    |               |         |              |             |
|          | P-50   | 6.36              | 2.32           | 16.42         | 4.6             | 8.77          | 3.18               | 2.1           | 2.44    | 2.06         | 4.34        |
| Q-12     | P-90   | 6.69              | 2.98           | 18.86         | 5.64            | 9.47          | 3.54               | 2.24          | 3.17    | 2.37         | 5.13        |
|          | Max    | 7.01              | 3.53           | 18.97         | 6.36            | 9.54          | 3.6                | 2.64          | 3.62    | 2.43         | 7.4         |
|          |        |                   |                |               |                 |               |                    |               |         |              |             |
|          | P-50   | 5.73              | 2.04           | 17.34         | 3.87            | 5.45          | 3.17               | 2.16          | 2.19    | 2.47         | 4.67        |
| Q-13     | P-90   | 6.92              | 2.19           | 19.91         | 5.6             | 5.9           | 3.74               | 2.34          | 3.07    | 3.46         | 5.29        |
|          | Max    | 6.95              | 2.78           | 20.39         | 5.8             | 6.35          | 3.96               | 2.52          | 5.46    | 4.51         | 5.34        |
|          |        |                   |                |               |                 |               |                    |               |         |              |             |
|          | P-50   | 6.19              | 2.29           | 16.35         | 5.81            | 8.84          | 3.13               | 2.09          | 2.27    | 2.13         | 4.38        |
| Q-14     | P-90   | 6.42              | 4.48           | 18.52         | 8.56            | 9.52          | 3.59               | 2.53          | 2.84    | 2.59         | 5.79        |
|          | Max    | 6.69              | 6.49           | 20.99         | 14.72           | 9.54          | 3.8                | 2.91          | 2.87    | 4.29         | 6.64        |
|          |        |                   |                |               |                 |               |                    |               |         |              |             |
|          | P-50   | 5.31              | 2.25           | 15.62         | 6.58            | 5.18          | 4.17               | 1.97          | 2.12    | 2.54         | 3.91        |
| Q-15     | P-90   | 5.72              | 3.31           | 18.28         | 8.42            | 5.37          | 4.77               | 8.02          | 4.75    | 2.84         | 4.9         |
|          | Max    | 6.04              | 3.45           | 19.57         | 8.63            | 5.41          | 5.68               | 8.2           | 5.82    | 2.84         | 6           |
|          |        |                   |                |               |                 |               |                    |               |         |              |             |
|          | P-50   | 17.68             | 5.9            | 37.57         | 124.03          | 6.28          | 9.22               | 3.54          | 75.95   | 66.59        | 21.11       |
| Q-16     | P-90   | 20.51             | 6.97           | 42.13         | 126.67          | 6.6           | 9.42               | 3.76          | 91.96   | 75.1         | 26.04       |
|          | Max    | 20.72             | 7.06           | 47.22         | 126.71          | 6.75          | 9.64               | 3.99          | 118.36  | 77.52        | 28.39       |
|          |        |                   |                |               |                 |               |                    |               |         |              |             |
|          | P-50   | 16.51             | 7.79           | 41.35         | 23.2            | 6.83          | 11.14              | 8.1           | 9.15    | 9.86         | 18.68       |
| Q-17     | P-90   | 17.74             | 10.38          | 47.66         | 26.49           | 7.27          | 11.81              | 8.23          | 10.37   | 11.68        | 20.98       |
|          | Max    | 18.43             | 12.43          | 49.04         | 32.98           | 8.02          | 12.13              | 8.23          | 12.2    | 12.22        | 22.43       |
|          |        |                   |                |               |                 |               |                    |               |         |              |             |
|          | P-50   | 10.43             | 5.47           | 23.17         | 8.74            | 3.44          | 7.46               | 6.12          | 6.26    | 5.77         | 11.03       |
| Q-18     | P-90   | 11.38             | 7.62           | 25.48         | 12.47           | 3.76          | 7.89               | 6.71          | 7.33    | 6.61         | 12.1        |
|          | Max    | 15.19             | 7.96           | 28.87         | 38.28           | 4.54          | 8.01               | 7.06          | 7.99    | 9.01         | 12.79       |
|          |        |                   |                |               |                 |               |                    |               |         |              |             |
|          | P-50   | 10.84             | 4.41           | 37.27         | 8.42            | 6.99          | 10.99              | 8.13          | 6.38    | 7.69         | 13.19       |
| Q-19     | P-90   | 11.2              | 9.94           | 42.08         | 24.29           | 7.2           | 11.21              | 8.59          | 8       | 13.26        | 14.88       |
|          | Max    | 11.74             | 10.99          | 44.95         | 37.83           | 7.59          | 11.28              | 8.6           | 8.67    | 14.55        | 19.17       |
|          |        |                   |                |               |                 |               |                    |               |         |              |             |
|          | P-50   | 31.54             | 18.18          | 77.33         | 132.14          | 7.4           | 5.12               | 8.19          | 92.99   | 4.16         | 141.51      |
| Q-20     | P-90   | 32.72             | 37.02          | 88.6          | 174.69          | 7.62          | 5.22               | 8.8           | 107.35  | 14.42        | 158.07      |
|          | Max    | 33.15             | 37.39          | 88.96         | 181.16          | 7.64          | 5.58               | 10.03         | 138.38  | 79.72        | 158.4       |

### RQ3: How verbose do different LLMs perform on a set of IT Operations tasks?

The below table presents the average token count values associated with each model as a summary of the results groupped by type of tasks:
| Model | SR Task | AR Task |
|:---:|:---:|:---:|
| | (0..1 tools) | (2+ tools) |
| Claude 3.5 Sonnet | 5991.2 | 42768 |
| Claude 3 Haiku | 6053.8 | 46536  |
| Claude 3 Opus | 6402.2 | 44648.5 |
| Mistral Largest | 6697.5 | 39877.7 |
| Mixtral 8X22B | 2822.4 | 3007.5 |
| Mistral Small 7B | 4642.1 | 4619.5 |
| GPT 3.5 Turbo | 3433.2 | 2722.2 |
| GPT 4-o | 3436.9 | 25965 |
| GPT 4-o Mini | 3480.5 | 20900.7 |
| GPT 4 Turbo | 3478.9 | 24308.7 |

The below table presents the detailed token count values for each query and LLM tested:

| Query \# | Claude 3.5 Sonnet | Claude 3 Haiku | Claude 3 Opus | Mistral Largest | Mixtral 8x22B | Mistral Small (7B) | GPT 3.5 Turbo | GPT 4-o | GPT 4-o Mini | GPT 4 Turbo |
| -------- | ----------------- | -------------- | ------------- | --------------- | ------------- | ------------------ | ------------- | ------- | ------------ | ----------- |
| Q-01     | 3056              | 6123           | 5101          | 6958            | 2857          | 2636               | 1813          | 1951    | 1919         | 2004        |
| Q-02     | 3265              | 6223           | 4752          | 7841            | 3061          | 2848               | 0\*           | 2174    | 2011         | 2097        |
| Q-03     | 6386              | 6021           | 6859          | 5488            | 2627          | 5685               | 3870          | 3856    | 3850         | 3982        |
| Q-04     | 6307              | 6093           | 6849          | 5597            | 2664          | 5582               | 3912          | 3801    | 3803         | 3915        |
| Q-05     | 7178              | 6203           | 3927          | 5697            | 2993          | 2799               | 4197          | 3083    | 2210         | 2386        |
| Q-06     | 6505              | 6173           | 6946          | 5798            | 2877          | 5673               | 3988          | 3942    | 3922         | 4018        |
| Q-07     | 6574              | 6230           | 6972          | 5792            | 2826          | 5712               | 3998          | 3944    | 3920         | 4025        |
| Q-08     | 6566              | 6200           | 6925          | 5727            | 2466          | 5677               | 3973          | 3927    | 3965         | 4022        |
| Q-09     | 6369              | 6174           | 6928          | 5621            | 2727          | 5617               | 3916          | 3813    | 3807         | 3930        |
| Q-10     | 6055              | 5809           | 6697          | 5239            | 2658          | 2623               | 3533          | 3423    | 3417         | 3527        |
| Q-11     | 6094              | 5829           | 6655          | 5247            | 2696          | 5267               | 3550          | 3437    | 3438         | 3554        |
| Q-12     | 6152              | 5890           | 6709          | 5277            | 3117          | 5286               | 3566          | 3457    | 3457         | 3570        |
| Q-13     | 6142              | 5867           | 6721          | 5279            | 2868          | 5290               | 3569          | 3460    | 3460         | 3575        |
| Q-14     | 6146              | 5886           | 6685          | 5278            | 3117          | 5287               | 3565          | 3457    | 3457         | 3570        |
| Q-15     | 6187              | 5949           | 6724          | 20770           | 2859          | 2729               | 3693          | 3587    | 5374         | 3696        |
| Q-16     | 34865             | 34023          | 35594         | 21463           | 2950          | 5062               | 0\*           | 25882   | 28049        | 20108       |
| Q-17     | 34822             | 25803          | 35862         | 33235           | 2994          | 5426               | 3601          | 11173   | 26946        | 21859       |
| Q-18     | 6877              | 6191           | 6985          | 5551            | 2745          | 5562               | 3788          | 3678    | 3679         | 3791        |
| Q-19     | 17319             | 12892          | 19398         | 19867           | 3021          | 5180               | 3638          | 11612   | 15648        | 11163       |
| Q-20     | 84066             | 113426         | 87740         | 84946           | 3065          | 2810               | 3650          | 55193   | 12960        | 44105       |

Q-02 and Q-16 were not resolved by the GPT 3.5 turbo model due to a model internal processing error (as reported by the integration library and visible in the results json files in this repository). The correlation between the token count and result
of the query is especially visible for these two queries as in RQ3 we see the count of tokens at zero, despite the model taking some times before an error is provided, as per the results from RQ2.

## What is Red Hat OpenShift

[Red Hat OpenShift Container Platform](https://www.redhat.com/en/technologies/cloud-computing/openshift/container-platform) (RHOCP) is a leading enterprise Kubernetes platform that
enables a cloud-like experience everywhere it's deployed. Whether it’s in the cloud, on-premise or at the edge, Red Hat OpenShift gives you the ability to choose where you build,
deploy, and run applications through a consistent experience (Source: [RedHat](https://www.redhat.com/en/technologies/cloud-computing/openshift)).

In other words, Red Hat OpenShift Container Platform is a private Platform as a Service (PaaS) for enterprises that run OpenShift on public clouds or on-premises infrastructure.
The OpenShift platform is a consistent hybrid cloud foundation for building and scaling containerized applications.
In other words, it is an open-source container orchestration platform for enterprises. It includes several container technologies, primarily the OpenShift container orchestration software,
which is based on the OKD open-source project. Red Hat OpenShift combines Kubernetes components with security features and productivity necessary for large enterprises and is especially
useful in hybrid cloud scenarios.

Red Hat OpenShift uses operators. [Operators](https://docs.openshift.com/container-platform/4.11/operators/understanding/olm-what-operators-are.html) are pieces of software that
ease the operational complexity of running another piece of software.

For reproducing the experiments completely the following list of operators are required:

- Red Hat OpenShift Data Foundation - we shall only use the Multicloud Gateway deployment option as we only need S3 compatible object bucket store.
- Red Hat OpenShift AI -(RHOAI) to run the ML experiments.
  - KubeRay - cluster deployment part of CodeFlare operator which part of RHOAI. This is optional and required only to accelare the [MLASP](https://github.com/eartvit/mlasp-on-ocp) model training.
- MLFlow - for storing the ML experiment results.
- CrunchyDB PostgreSQL operator - support for MLFlow (required to store experiment metadata information and other MLFlow related metadata).
- Tekton - for running automated pipeline workloads (in our case for syntetic data generation for the ML experiments).
- Prometheus - for collecting metrics required for ML experiments.

Visual guides for deployment that may be used for this purpose are available in a similars project [here](https://youtu.be/Eh3azO9tRJw) and [here](https://github.com/eartvit/llm-on-ocp).
