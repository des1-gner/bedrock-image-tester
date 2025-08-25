# Bedrock Image Payload Limits - Empirical Analysis

## Overview

This repository contains empirical testing and analysis of Amazon Bedrock's image payload limits when using the Converse API. The testing was conducted to understand the practical limitations when sending multiple images in a single API request.

## Background

While Amazon Bedrock's documentation specifies individual image limits (up to 20 images, each ≤3.75MB, ≤8000px dimensions), there are additional payload size restrictions that can cause requests to fail with "Input is too long for requested model" errors even when individual images meet the specified criteria.

## Key Findings

### Payload Size Limit
- **Total request payload limit**: 20MB (as documented for InvokeModel/InvokeModelWithResponseStream)
- **Applies to**: Converse/ConverseStream APIs as well, despite not being explicitly documented
- **Enforcement**: Payload limit is enforced before model context window limits

### Test Results Summary

| Image Size | Copies | Result | Input Tokens | Notes |
|------------|--------|--------|--------------|-------|
| 0.85MB | 20 | ✅ Success | 30,811 | Within payload limits |
| 1.31MB | 20 | ❌ Failed | - | ~26MB total exceeds limit |
| 2.48MB | 7 | ✅ Success | 10,804 | Within payload limits |
| 2.48MB | 8 | ❌ Failed | - | ~20MB+ total exceeds limit |

## Technical Details

### Why Requests Fail
1. **Base64 encoding overhead**: Increases image data size by ~33%
2. **JSON payload structure**: Additional metadata and formatting
3. **Multiple image accumulation**: Total payload grows with each image
4. **API enforcement**: 20MB limit enforced before context window processing

### Factors Affecting Payload Size
- Original image file size
- Base64 encoding expansion
- JSON request structure overhead
- Number of images in request

## Recommendations

### For Multiple Image Processing
1. **Compress images** while maintaining acceptable quality
2. **Resize images** to smaller dimensions if suitable for use case
3. **Process in batches** (10-15 images per request instead of 20)
4. **Monitor total payload size** rather than just individual image limits

## Code Structure

The notebook contains:
- **Image testing functions** adapted from AWS documentation
- **Payload size analysis** with different image sizes and quantities
- **Token usage tracking** for cost estimation
- **Comprehensive results summary**

## Files

- `empirical_analysis_on_bedrock_image_limits.ipynb` - Main testing notebook
- `empirical_analysis_on_bedrock_image_limits.pdf` - Exported results for sharing
- Test images (various sizes for validation)

## Usage

The code is adapted from [AWS Bedrock conversation inference examples](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference-examples.html) and can be easily modified to test different models or image configurations.

## References

- [Bedrock Message API Documentation](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Message.html)
- [Claude Messages Request/Response Parameters](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-anthropic-claude-messages-request-response.html)
- [Bedrock Conversation Inference Examples](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference-examples.html)

## Note

Testing was conducted using publicly available images from Pexels for validation purposes. The same image was sent multiple times for simplicity, but results apply to multiple different images as well.
