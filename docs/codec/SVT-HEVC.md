# SVT-HEVC Code Investigation

- [SVT-HEVC github repo](https://github.com/OpenVisualCloud/SVT-HEVC)
- [my repo with latency profile](https://github.com/spartazhc/SVT-HEVC)
- Read [white paper](https://01.org/sites/default/files/documentation/svt_aws_wp.pdf) of Scalable Video Technology (SVT)  first.

## Initiation

To learn how SVT works you should understand how the program initiates.

- `EbEncHandle.c` is the key file in initiation, and `SystemResource` is the key in multi-thread.
- Structures need to learn in `EbSystemResourceManager.c`
  - `EbSystemResource_t`
  - `EbObjectWrapper_t`
  - `EbMuxingQueue_t`
  - `EbCircularBuffer_t`
  - `EbFifo_t`

```c
1. EbAppMain.c: main
   
2. EbAppContext.c: InitEncoder [important]
    // STEP 1: Call the library to construct a Component Handle
    return_error = EbInitHandle(&callbackData->svtEncoderHandle, callbackData, &callbackData->ebEncParameters);
    // STEP 3: Copy all configuration parameters into the callback structure
    return_error = CopyConfigurationParameters(
                    config,
                    callbackData,
                    instanceIdx);
    // STEP 4: Send over all configuration parameters
    // Set the Parameters
    return_error = EbH265EncSetParameter(
                       callbackData->svtEncoderHandle,
                       &callbackData->ebEncParameters);
    // STEP 5: Init Encoder
    return_error = EbInitEncoder(callbackData->svtEncoderHandle);
    ///************************* LIBRARY INIT [END] *********************///
    ///********************** APPLICATION INIT [START] ******************///
    // STEP 6: Allocate input buffers carrying the yuv frames in
    return_error = AllocateInputBuffers(
        config,
        callbackData);
    // STEP 7: Allocate output buffers carrying the bitstream out
    return_error = AllocateOutputBuffers(
        config,
        callbackData);
    // STEP 8: Allocate output Recon Buffer
    return_error = AllocateOutputReconBuffers(
        config,
        callbackData);
	// Allocate the Sequence Buffer
    if (config->bufferedInput != -1) {
        // Preload frames into the ram for a faster yuv access time
        PreloadFramesIntoRam(config);
    } else {
        config->sequenceBuffer = 0;
    }

3. EbEncHandle.c: LoadDefaultBufferConfigurationSettings
    // define all resources' (buffer / fifo / thread ...) InitCount 
    
4. EbEncHandle.c: EbInitEncoder [important]
	/** 
	 * Initiate following:
     * 1. Sequence Control Set
     * 2. Picture Control Set: Parent
     * 3. Picture Control Set: Child
     * 4. Picture Buffers
     * 5. System Resource Managers & Fifos
     * 6. App Callbacks
     * 7. Contexts
     * 8. Thread Handles
     */
5. Thread list:
	/**
	 * 1. ResourceCoordinationKernel
	 * 2. PictureAnalysisKernel
	 * 3. PictureDecitionKernel
	 * 4. MotionEstimationKernel
	 * 5. InitialRateControlKernel
	 * 6. SourceBasedOperationsKernel
	 * 7. PictureManagerKernel
	 * 8. RateControlKernel
	 * 9. ModeDecisionConfigurationKernel
	 * 10. EncDecKernel
	 * 11. EntropyCodingKernel
	 * 12. PacketizationKernel
	 */
```

## Resource Coordination in XXXKernels

From Initiation, we know that different layer pass object by inter result fifos which is defined in contexts.

- Basically you are doing like:
  1. get a full object(produced by the layer before you)
  2. get what you need from this object
  3. do process
  4. get a empty object and put your result into it
  5. release input object to empty object queue
  6. post full object to full object queue
  7. goto 1

- Take `EbMotionEstimationProcess.c` as example:

```c
// Get Input Full Object
EbGetFullObject(
    contextPtr->pictureDecisionResultsInputFifoPtr,
    &inputResultsWrapperPtr);
...
// Get Empty Results Object
EbGetEmptyObject(
    contextPtr->motionEstimationResultsOutputFifoPtr,
    &outputResultsWrapperPtr);
outputResultsPtr = (MotionEstimationResults_t*)outputResultsWrapperPtr->objectPtr;
...

// Release the Input Results
EbReleaseObject(inputResultsWrapperPtr);
// Post the Full Results Object
EbPostFullObject(outputResultsWrapperPtr);
```

## Process Pipeline

| Kernel                          | In   | Out       | sidepath                |      |
| ------------------------------- | ---- | --------- | ----------------------- | ---- |
| ResourceCoordinationKernel      | pic  | pic       |                         |      |
| PictureAnalysisKernel           | pic  | pic       |                         |      |
| PictureDecitionKernel           | pic  | seg       |                         |      |
| MotionEstimationKernel          | seg  | seg       |                         |      |
| InitialRateControlKernel        | seg  | pic       |                         |      |
| SourceBasedOperationsKernel     | pic  | pic       |                         |      |
| PictureManagerKernel            | pic  | pic       |                         |      |
| RateControlKernel               | pic  | pic       |                         |      |
| ModeDecisionConfigurationKernel | pic  | tile      |                         |      |
| EncDecKernel                    | tile | seg       | to itself / PicMgr      |      |
| EntropyCodingKernel             | seg  | pic / seg | to RateControl          |      |
| PacketizationKernel             | pic  | pic       | to RateControl / PicMgr |      |

```c
// The reason of sidepath is because following task types
// in RateControlTasks_t
typedef enum RATE_CONTROL_TASKTYPES {
    RC_PICTURE_MANAGER_RESULT,
    RC_PACKETIZATION_FEEDBACK_RESULT,
    RC_ENTROPY_CODING_ROW_FEEDBACK_RESULT,
    RC_INVALID_TASK
} RATE_CONTROL_TASKTYPES;

// in PictureDemuxResults_t 
typedef enum EB_PIC_TYPE {
    EB_PIC_INVALID = 0,
    EB_PIC_INPUT = 1,
    EB_PIC_REFERENCE = 2,
    EB_PIC_FEEDBACK = 3
} EB_PIC_TYPE;
```

