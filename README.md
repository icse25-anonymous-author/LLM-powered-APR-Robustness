# Exploring and Lifting the Robustness of LLM-powered Automated Program Repair with Metamorphic Testing

## Datasets:
Two datasets are employed as our test subjects:
* Defects4J
* QuixBugs

## Models:
We choose four recent LLMs: Mistral Large, LLaMA3-70B, LLaMA3-8B, and CodeGemma-7B. The parameter settings and prompt templates for all LLMs reviewed are fixed the same, as follows:
* temperature=0
* top_p=0.8
* max_tokens=1024
* stream=True
* Prompt template(An enxample):
```python
prompt = f"""
    The following Java code contains a bug. Your task is to fix the bug and provide the corrected code only. Do not include any other text in your response.
    Buggy code:
    {
    package java_programs;
    import java.util.*;
    import java.util.ArrayDeque;
    
    /*
     * To change this template, choose Tools | Templates
     * and open the template in the editor.
     */
    
    /**
     *
     * @author derricklin
     */
    public class BREADTH_FIRST_SEARCH {
    
        public static Set<Node> nodesvisited = new HashSet<>();
    
        public static boolean breadth_first_search(Node startnode, Node goalnode) {
            Deque<Node> queue = new ArrayDeque<>();
            queue.addLast(startnode);
    
            nodesvisited.add(startnode);
    
            while (true) {
                Node node = queue.removeFirst();
    
                if (node == goalnode) {
                    return true;
                } else {
                    for (Node successor_node : node.getSuccessors()) {
                        if (!nodesvisited.contains(successor_node)) {
                            queue.addFirst(successor_node);
                            nodesvisited.add(successor_node);
                        }
                    }
                }
            }
            /**
             * The buggy program always drops into while(true) loop and will not return false
             * Removed below line to fix compilation error
             */
            //return false;
        }    
    }
}
    Fixed code:
    """
```
The output is(Fixed code):
```java
package java_programs;
import java.util.*;
import java.util.ArrayDeque;

/*
 * To change this template, choose Tools | Templates
 * and open the template in the editor.
 */

/**
 *
 * @author derricklin
 */
public class BREADTH_FIRST_SEARCH {

    public static Set<Node> nodesvisited = new HashSet<>();

    public static boolean breadth_first_search(Node startnode, Node goalnode) {
        Deque<Node> queue = new ArrayDeque<>();
        queue.addLast(startnode);

        nodesvisited.add(startnode);

        while (!queue.isEmpty()) {
            Node node = queue.removeFirst();

            if (node == goalnode) {
                return true;
            } else {
                for (Node successor_node : node.getSuccessors()) {
                    if (!nodesvisited.contains(successor_node)) {
                        queue.addLast(successor_node);
                        nodesvisited.add(successor_node);
                    }
                }
            }
        }
        return false;
    }

}
```

## Code:
This repository includes the following code:

* Code for using AST to implement nine kinds of MRs: Located in the `AST` directory.
* Code for LLM-based code patch generation: Located in the `LLM_test` directory.
* Verification of the correctness of patches generated by LLM: Located in the `LLM_validation` directory.
* The acquisition of experimental results: Located in the `evaluation` directory.
* Code for training a CodeT5-based code editing model aiming at improving code readability: Located in the `codet5` directory.

## Quick Start

### LLM code patch generation.

This step is designed to test the LLM's repair performance on the original datasets to build base samples. Navigate to the `LLM_test` directory. Before running the code, please perform different operations according to diffenrent datasets:

#### Defects4J: 

- Configuration Variables

  The corresponding code file is `LLM_test_Defects4J.py`. Before running the code, please modify the following variables according to your requirements:

  * `input_file`: Specify the file path of the input data of Defects4J, namely, `single_function_repair.json`, which located in the same directory.
  * `keylist`:  A list containing multiple keys can be used. You can fill in a certain number of API keys according to your used models and actual needs.
  * `llm_models`: A list of different LLM names. You can change the contents of this list to test multiple LLMS.
  * `output_dir`: You can change the field name according to the actual situation, as the output directory of the test results, for example, `fixed_code_mistrallarge`.

- Running the Code

  Ensure that you have correctly set the variables mentioned above.

  Run the corresponding Python script to perform the task, for example:
  ```python
  $ python LLM_test_Defects4J.py
  ```

#### QuixBugs: 

- Configuration Variables

  The corresponding code file is `LLM_test_QuixBugs.py`. Before running the code, please modify the following variables according to your requirements:

  * `keylist`:  A list containing multiple keys can be used. You can fill in a certain number of API keys according to your used models and actual needs.
  * `buggy_dir`: The path to the directory where the Java buggy program is stored in the QuixBugs dataset, for example, `QuixBugs/java_programs`.
  * `models`: A dict of different LLM names and thier model path. You can change the contents of this dict to test multiple LLMS.
  * `fixed_dir`: You can change the field name according to the actual situation, as the output directory of the generated results, for example, `fixed_code_mistrallarge`.

- Running the Code

  Ensure that you have correctly set the variables mentioned above.

  Run the corresponding Python script to perform the task, for example:
  ```python
  $ python LLM_test_QuixBugs.py
  ```

#### Pre-processing of generated patches

- Configuration Variables

 The corresponding code file is `pre-processing.py`. Before running the code, please modify the following variables according to your requirements:

  * `source_folder`: The output directory of the generated results in the previous step, for example, `fixed_code_mistrallarge`.
  * `target_folder`: The output directory of the results after pre-processing, for example, `fixed_code_mistrallarge_pre`.

- Running the Code

  Ensure that you have correctly set the variables mentioned above.

  Run the corresponding Python script to perform the task, for example:
  ```python
  $ python pre-processing.py
  ```

### Verify that the LLM generated the correct patch.

 Navigate to the `LLM_validation` directory.

#### Defects4J: 

- Configuration Variables

  The corresponding code file is `validate_Defects4J.py`. Before running the code, please modify the following variables according to your requirements:

  * `main_folder`: This directory holds the code patches generated by the LLM and ensures that all code patches have been pre-processed.
  * `loc_folder`:  This directory corresponds to the path of `location` directory in the Defects4J raw dataset, for example, `Defects4j/location`.
  * `input_file`: Specify the file path of the input data of Defects4J, namely, `single_function_repair.json`, which located in `LLM_test` directory.
  * `output_result `: You can change the field name according to the actual situation, as the output directory of the validation results, for example, `validation_result`.
  
- Running the Code

  Ensure that you have correctly set the variables mentioned above.

  Run the corresponding Python script to perform the task, for example:
  ```python
  $ python validate_Defects4J.py
  ```

#### QuixBugs: 

- Configuration Variables

  The corresponding file is `validate_QuixBugs.sh`. Before running the script, please modify the following variables according to your requirements:

  * `LLM_DIR`: The path to the output directory of the results after pre-processing, for example, `fixed_code_mistrallarge_pre`. 
  * `JAVA_PROGRAMS_DIR`: The path to the directory where the Java buggy program is stored in the QuixBugs dataset, for example, `QuixBugs/java_programs`.
  * `RESULT_FILE`: The path to the file where the test results(successfully repaired) will be stored.
  * `FAILURE_FILE`: The path to the file where the test results(unsuccessfully repaired) will be stored.
  
- Running the Script

  Ensure that you have correctly set the variables mentioned above.

  Run the corresponding shell script to perform the task, for example:
  ```shell
  $ ./ validate_QuixBugs.sh
  ```
#### Construction of base samples
After the above process, we obtain 60 base samples from each dataset, where 15 samples for each LLM, namely Defect4J<sub>base</sub> and QuixBugs<sub>base</sub>. Details of the base sample construction results
are as follows:
![数据集采样结果](https://github.com/user-attachments/assets/1d25ea25-3347-452b-a844-b967c5dd0890)

### Implemention of Nine MRs Based on AST.
Navigate to the `AST` directory and execute the following code to implemnt MRs:

#### Defects4J: 

- Configuration Variables

  The corresponding code file is `per_Defects4J.py`. Before running the code, please modify the following variables according to your requirements:

  * `input_directory`: The directory of the results after pre-processing, for example, `fixed_code_mistrallarge_pre`.
  * `java_class_path`: The path to the Java library javaparser, for example, `javaparser-core-3.25.8.jar`, which located in the same directory.
  * `output_directory`: The path where the perturbed code patch is stored, for example, `perturbated_result_Defects4J`.
  * `filenames` : The path to the directory of the successful repaired samples/unsuccessful repaired samples.

- Running the Code

  Ensure that you have correctly set the variables mentioned above.

  Run the corresponding Python script to perform the task, for example:
  ```python
  $ python per_Defects4J.py
  ```
  
#### QuixBugs: 

  The corresponding code file is `per_QuixBugs.py`. Before running the code, please modify the following variables according to your requirements:

  * `input_directory`: The path to the directory where the Java buggy program is stored in the QuixBugs dataset, for example, `QuixBugs/java_programs`.
  * `java_class_path`: The path to the Java library javaparser, for example, `javaparser-core-3.25.8.jar`, which located in the same directory.
  * `output_directory`: The path where the perturbed code patch is stored, for example, `perturbated_result_QuixBugs`.

- Running the Code

  Ensure that you have correctly set the variables mentioned above.

  Run the corresponding Python script to perform the task, for example:
  ```python
  $ python per_QuixBugs.py
  ```

### LLM code patch generation after perturbation.

This step is designed to test the LLM's repair performance on the perturbated samples. Navigate to the `LLM_test` directory. Before running the code, please perform different operations according to diffenrent datasets:

#### Defects4J: 

- Configuration Variables

  The corresponding code file is `LLM_test_Defects4J_after.py`. Before running the code, please modify the following variables according to your requirements:

  * `input_file`: Specify the file path of the input data of Defects4J, namely, `single_function_repair.json`, which located in the same directory.
  * `keylist`:  A list containing multiple keys can be used. You can fill in a certain number of API keys according to your used models and actual needs.
  * `projects_gemma`,`projects_8b`,`projects_70b`,`projects_mistral`: Project names in Defects4J that the corresponding LLM is able to fix successfully.
  * `model_project_mapping`: A dict that maps the LLM to its corresponding projects.
  * `model_name_mapping`: A dict of different LLM names and thier model path. You can change the contents of this dict to test multiple LLMS.
  * `base_dir`: The path where the perturbed code patch is stored.
  * `workers`； The hyperparameter of sample extraction.

- Running the Code

  Ensure that you have correctly set the variables mentioned above.

  Run the corresponding Python script to perform the task, for example:
  ```python
  $ python LLM_test_Defects4J_after.py
  ```

#### QuixBugs: 

- Configuration Variables

  The corresponding code file is `LLM_test_QuixBugs_after.py`. Before running the code, please modify the following variables according to your requirements:

  * `keylist`:  A list containing multiple keys can be used. You can fill in a certain number of API keys according to your used models and actual needs.
  * `projects_gemma`,`projects_8b`,`projects_70b`,`projects_mistral`: Project names in QuixBugs that the corresponding LLM is able to fix successfully.
  * `model_project_mapping`: A dict that maps the LLM to its corresponding projects.
  * `model_name_mapping`: A dict of different LLM names and thier model path. You can change the contents of this dict to test multiple LLMS.
  * `base_dir`: The path where the perturbed code patch is stored.

- Running the Code

  Ensure that you have correctly set the variables mentioned above.

  Run the corresponding Python script to perform the task, for example:
  ```python
  $ python LLM_test_QuixBugs_after.py
  ```

### Verify that the LLM generated the correct patch after perturbation.

 Navigate to the `LLM_validation` directory.

#### Defects4J: 

- Configuration Variables

  The corresponding code file is `validate_Defects4J_after.py`. Before running the code, please modify the following variables according to your requirements:

  * `main_folder`: This directory holds the code patches generated after perturbation by the LLM and ensures that all code patches have been pre-processed.
  * `loc_folder`:  This directory corresponds to the path of `location` directory in the Defects4J raw dataset, for example, `Defects4j/location`.
  * `input_file`: Specify the file path of the input data of Defects4J, namely, `single_function_repair.json`, which located in `LLM_test` directory.
  * `output_result `: You can change the field name according to the actual situation, as the output directory of the validation results, for example, `validation_result`.
  
- Running the Code

  Ensure that you have correctly set the variables mentioned above.

  Run the corresponding Python script to perform the task, for example:
  ```python
  $ python validate_Defects4J_after.py
  ```

#### QuixBugs: 

- Configuration Variables

  The corresponding file is `validate_QuixBugs_after.sh`. Before running the script, please modify the following variables according to your requirements:

  * `LLM_DIR`: The path to the output directory of the results after pre-processing. 
  * `JAVA_PROGRAMS_DIR`: The path to the directory where the Java buggy program is stored in the QuixBugs dataset, for example, `QuixBugs/java_programs`.
  * `RESULT_FILE`: The path to the file where the test results(successfully repaired) will be stored.
  * `ERROR_FILE`: The path to the file where the test results(unsuccessfully repaired) will be stored.
  
- Running the Script

  Ensure that you have correctly set the variables mentioned above.

  Run the corresponding shell script to perform the task, for example:
  ```shell
  $ ./ validate_QuixBugs_after.sh
  ```

### Experimental results

#### Experimental results for different RQs:

##### Defects4J

- Configuration Variables

  The corresponding file is `analysis_Defects4J.py`. Before running the script, please modify the following variables according to your requirements:

  * `root_directory`: The path of the results of Defects4J obtained from the above process. 
  * `result_file`: The results of different RQs of Defects4J.
  
- Running the Script

  Ensure that you have correctly set the variables mentioned above.

  Run the corresponding script to perform the task, for example:
  ```shell
  $ python analysis_Defects4J.py
  ```

##### QuixBugs

- Configuration Variables

  The corresponding file is `analysis_QuixBugs.py`. Before running the script, please modify the following variables according to your requirements:

  * `files_to_read`: List of files to read, which are obtained from the above process. 
  * `result_file`: The results of different RQs of QuixBugs.
  
- Running the Script

  Ensure that you have correctly set the variables mentioned above.

  Run the corresponding script to perform the task, for example:
  ```shell
  $ python analysis_QuixBugs.py
  ```

#### Edit the distance calculation.

Depending on the specific task, you can modify the path contents in the following Python files as needed:

* `disatnce_Defects4J.py`: Calculate the edit distance for samples from Defects4J.
* `disatnce_QuixBugs.py`: Calculate the edit distance for samples from QuixBugs.
* `disatnce_average.py`: Calculate the average edit distance for both datasets.
* `distance_zero`: Calculate the ratio of edit distance to zero.

### Improvement

Train a CodeT5-based code editing model aiming at improving buggy code readability.

#### Step1. Obtain training data.

- Configuration Variables

  Navigate to the `codet5` directory. The corresponding file is `generate_train.py`. Before running the script, please modify the following variables according to your requirements:

  * `disturb_fail_dir`: The path to the directory where the test results(unsuccessfully repaired) will be stored.
  * `repair_json_path`: Specify the file path of the input data of Defects4J, namely, `single_function_repair.json`, which located in `LLM_test` directory.
  * `output_json_path`: The path to the output file.
  
- Running the Script

  Ensure that you have correctly set the variables mentioned above.

  Run the corresponding script to perform the task, for example:
  ```shell
  $ python generate_train.py
  ```

#### Step2. Model training.

- Configuration Variables

  The corresponding file is `train.py`. You can adjust the contents of this file according to your devices and resources.
  
- Running the Script

  Run the corresponding script to perform the task, for example:
  ```shell
  $ python train.py
  ```

#### Step3. Model testing.

- Configuration Variables

  The corresponding file is `test.py`. You can adjust the contents of this file according to your actual demands.
  
- Running the Script

  Run the corresponding script to perform the task, for example:
  ```shell
  $ python test.py
  ```


## Contribution
Contributions to this project through Pull Requests or Issues are welcome.

## License
This project is licensed under the `MIT` License.
# LLM-powered-APR-Robustness
