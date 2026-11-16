## Static Analysis Alarm Dataset

This dataset contains detailed static analysis alarm data, presented in JSON format. Each JSON file corresponds to a static analysis result for a particular project. Each item within the JSON file represents an individual alarm and includes detailed information such as the associated static analysis result, code context, bug confirmation source, and intermediate result derived from the static analyzer SVF https://github.com/SVF-tools/SVF.

### File Structure
The dataset is organized into multiple JSON files, each of which represents an individual analysis result corresponding to a specific project. These files are sorted into two directories: 'VF', representing the outcomes of direct Value-Flow Analysis, and 'OFB', denoting the results of Out-of-Bound Analysis.

```
dataset/
├── VF/
    ├── project_1.json
    ├── project_2.json
    ├── project_3.json
    ├── ...
├── OFB/
    ├── project_1.json
    ├── project_2.json
    ├── project_3.json
    ├── ...
```

### Data Format
Each JSON file contains fields corresponding to the following types of information:

1. Static analysis result: the original result of the analysis tool SVF.

- FullMessage: the analysis output of SVF

```
{
	"FullMessage": "NeverFree: memory allocation at : ...",
}
{
	"FullMessage": "PartialLeak: memory allocation at : ...",
}
```

2. Code context: The code snippet or context where the alarm was raised.

- (Caller/Callee)AllocFuncName: the caller/callee function of the memory allocation site
- (Forward/Backward)SliceFuncNames: functions encountered during forward/backward value-flow traversing
- (Forward/Backward)SliceVarNames: variables encountered during forward/backward value-flow traversing

```
{
    "CalleeAllocFuncName": "malloc",
    "CallerAllocFuncName": "stbi__convert_format16"
	"ForwardSliceFuncNames": "load_image_stb stbi__convert_format16 stbi__load_and_postprocess_16bit stbi__load_and_postprocess_8bit stbi__load_main stbi__loadf_main stbi__psd_load stbi_load stbi_load_16 stbi_load_16_from_callbacks stbi_load_16_from_memory stbi_load_from_callbacks stbi_load_from_file stbi_load_from_file_16 stbi_load_from_memory ",
     "ForwardSliceVarNames": "data dest good orig out result row0 row1 ",
     "BackwardSliceFuncNames": "load_image_stb stbi__convert_format16 stbi__load_and_postprocess_16bit stbi__load_and_postprocess_8bit stbi__load_main stbi__loadf_main stbi__psd_load stbi_load ",
      "BackwardSliceVarNames": "data good out result ",
}
```

3. Bug confirmation source: The source or reference that confirms the presence of the bug.

- Bug: whether the alarm corresponds to a real bug
  - `3` means we label corresponding alarm as true alarm, other values or no such field means the corresponding alarm is false alarm
- BugReason: the reason why it is a real bug and the corresponding reference (if any)

```
{
	"Bug": 3,
    "BugReason": "make_lstm_layer related functions contains many possible memory leaks that are fixed later: fixed version: https://github.com/AlexeyAB/darknet/commit/5bc62b14e06a3fcfda4e3a19fba77589920eddee "
}
```

4. Static analysis inner information: Intermediate analysis result obtained by our static analyzer instrumentation.

- Pointer properties
  - Count of casts: the number of cast instructions which may introduce problem
    - (Forward/Backward)CastAllCount: all cast number
    - (Forward/Backward)CastIntToPtrCount: number of int to pointer cast
    - (Forward/Backward)CastPtrToIntCount: number of pointer to int cast
    - (Forward/Backward)CastLossBitCount: number of cast that may cause bit loss
  - In/out degrees of value-flow nodes: reflecting the approximations of analysis result
    - (Forward/Backward)DstNodeInDegMap: in degree of dest node on edge
    - (Forward/Backward)DstNodeOutDegMap: out degree of dest node on edge
    - (Forward/Backward)SrcNodeInDegMap: in degree of source node on edge
    - (Forward/Backward)SrctNodeOutDegMap: out degree of source node on edge
    - (Forward/Backward)DstNodeIndEdgeInDegMap: in degree of dest node on indirect value-flow edge
    - (Forward/Backward)DstNodeIndEdgeOutDegMap: out degree of dest node on indirect value-flow edge
    - (Forward/Backward)SrcNodeIndEdgeInDegMap: in degree of source node on indirect value-flow edge
    - (Forward/Backward)SrcNodeIndEdgeOutDegMap: out degree of source node on indirect value-flow edge
    - (Forward/Backward)FunctionInDegMap: in degree of a function
    - (Forward/Backward)FunctionOutDegMap: out degree of a function
  - GEP information: reflecting the pointer arithmetic pattern of a value-flow path
    - (Forward/Backward)GEPAllCount: number of GEP instructions
    - (Forward/Backward)GEPAllArrayCount: number of GEP instructions from a typed array with length information
  - Indirect edge information: edges derived from approximated pointer analysis
    - (Forward/Backward)IndEdgeCount(Call/Intra/Ret): number of indirect edges in call/intra/return instruction
    - (Forward/Backward)IndTypeFuncMisMatch: number of indirect edges with potentially incompatible type of source and sink node
  - Other information
    - (Forward/Backward)IndPSetSizes: the size of points-to set
    - Repeated(Forward/Backward)Edges: repeated edges encountered during traversing, which may indicate loop or recursion
    - Repeated(Forward/Backward)IndirectEdges: repeated indirect edges encountered during traversing, which may indicate loop or recursion and points-to result

- Condition
  - CondCompareTo(Constant/NonConst/Null/Zero)Num: number of conditional branch with different comparison pattern, e.g. Null means instruction like ```p==nullptr```
  - CondCondInnerNum: numer of conditional branch with inner condition
  - Cond(Lines/Columns/FuncNames/FileNames): the location of the conditional instruction
  - (Forward/Backward)LoopUniqueCount: unique loops encountered during traversing
  - (Forward/Backward)LoopConstInitIVCount: loop with const induction variable
  - (Forward/Backward)LoopPassInstCount: the number of instructions the loop traversed
  - (Forward/Backward)LoopCanonicalCount: the number of canonical loop visited
  - (Forward/Backward)Loop(DepthInfo/NumBlocksInfo/NumOfBranches)：reflecting the complexity of loop
  - (Forward/Backward)Loop(FileInfo/LineInfo/NameInfo): the location of loop

Note:

- We have three types of countable numbers
  - single number: e.g. `"RepeatForwardAllEdges": 5,`
  - multiple numbers: e.g. `"ForwardSrcNodeOutDegMap": "1x1 2x4 ",`
    - there is 1 src-node has the out-degree of 1
    - there are 4 src-nodes have the out-degree of 2
    - etc.
- The meaning of these countable numbers is consistent on all presented data

### Usage
This dataset can be used for various purposes, including:

- Research and analysis of static analysis alarm.
- Development and evaluation of static analysis tools.
- Machine learning or data mining projects related to software bugs and vulnerabilities.

### Citation
If you use this dataset in your research or project, please consider citing it using the following citation format:

Author, A. (Year). Static Analysis Alarm Dataset [Data set]. Retrieved from [URL]

### License
This dataset is released under the Creative Commons Attribution 4.0 International License. Please refer to the LICENSE file for more information.
