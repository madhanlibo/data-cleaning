Breakdown of standard **data cleaning steps** for various data types:

---

### **1. Tabular Data Cleaning Steps**

Common for structured data like spreadsheets or CSV files.

* **Remove Duplicates:** Drop duplicate rows.
* **Handle Missing Values:** Fill (mean, median, mode, forward/backward fill) or drop.
* **Fix Data Types:** Convert columns to appropriate data types (e.g., int, float, datetime).
* **Standardize Categorical Values:** Normalize values (e.g., "M" vs "Male").
* **Outlier Detection:** Identify and treat anomalies using z-scores, IQR, etc.
* **Normalize/Scale Data:** Useful for ML (e.g., MinMaxScaler, StandardScaler). -> Usually done after train test split 
* **Remove Irrelevant Columns:** Drop features not relevant to your analysis.
* **Correct Typos/Formatting:** Fix inconsistent capitalization, date formats, etc.

---

### **2. Text Data Cleaning Steps**

Used in NLP, chatbots, sentiment analysis, etc.

* **Lowercasing:** Standardize text for comparison.
* **Remove Punctuation/Special Characters:** Clean out non-alphanumeric characters.
* **Tokenization:** Split text into words or phrases.
* **Remove Stop Words:** Remove common words (e.g., “the,” “and”).
* **Stemming/Lemmatization:** Reduce words to their base form.
* **Spell Correction:** Fix typos and spelling errors.
* **Remove HTML Tags/URLs:** Useful for web-scraped data.
* **Remove Non-English or Unwanted Language Tokens**

---

### **3. Image Data Cleaning Steps**

Preprocessing for CV models, classification, etc.

* **Resize Images:** Standardize dimensions.
* **Normalize Pixel Values:** Scale (e.g., 0-1 or -1 to 1).
* **Remove Corrupt Images:** Detect unreadable or broken files.
* **Check for Duplicates:** Use hashing to identify and remove.
* **Denoising:** Apply filters like Gaussian blur or median filter.
* **Convert Formats:** Standardize to one format (e.g., JPEG, PNG).
* **Augmentation Check:** If augmenting, ensure realistic transformations.

---

### **4. Video Data Cleaning Steps**

For surveillance, motion detection, etc.

* **Extract Frames:** Sample key frames if needed.
* **Remove Corrupt Files/Frames:** Skip unreadable or corrupted segments.
* **Standardize Frame Rate/Resolution**
* **Audio-Video Sync Check:** Ensure audio matches video (if relevant).
* **Trim Unnecessary Segments:** Cut intros, blank screens, or end credits.
* **Label Correction:** Verify accuracy of video metadata or annotations.

---

### **5. Time Series Data Cleaning Steps**

Used in forecasting, anomaly detection, etc.

* **Handle Missing Timestamps/Values:** Interpolate, forward/backward fill.
* **Detect & Remove Outliers:** Z-score, IQR, or domain-specific thresholds.
* **Resample:** Align to consistent frequency (e.g., hourly, daily).
* **Time Zone Normalization:** Convert all data to a common timezone.
* **Detrend/Decompose:** Separate trend, seasonality, and residuals.
* **Smooth Data:** Moving averages or exponential smoothing.
* **Check for Duplicates or Gaps in Time Index**

---

### **6. Telemetry Data Cleaning Steps**

Telemetry data is time-stamped data from sensors (IoT, cars, satellites).

* **Parse Time Properly:** Convert to datetime format.
* **Remove Sensor Noise:** Use filters (Kalman, Savitzky-Golay).
* **Remove or Impute Missing Sensor Readings**
* **Synchronize Multi-Sensor Data:** Align timestamps across sensors.
* **Validate Ranges:** Check values are within expected sensor thresholds.
* **Check for Drift or Calibration Issues:** Adjust if hardware drift is detected.
* **Downsample/Compress:** Reduce frequency without losing signal integrity.

---


### **7. Audio Data**

Used in speech recognition, music classification, etc.

* **Noise Reduction:** Remove background or static noise.
* **Trim Silence:** Cut out silence from start/end.
* **Normalize Volume:** Standardize audio levels.
* **Sample Rate Conversion:** Ensure consistency (e.g., 44.1kHz).
* **Format Conversion:** WAV, MP3, etc.
* **Segmentation:** Split long audio files into labeled segments.
* **Detect Corrupt Files:** Ensure no unreadable or partially saved files.

---

### **8. Graph Data**

Used in social networks, recommendation systems, etc.

* **Remove Isolated Nodes:** If not needed.
* **Check for Loops/Multiple Edges:** Simplify or clean structure.
* **Normalize Edge Weights:** Scale weights between a defined range.
* **Prune Low-Value Connections:** Remove edges below a threshold.
* **Validate Graph Connectivity:** Ensure nodes are reachable as expected.
* **Standardize Node/Edge Attributes:** Handle missing or inconsistent metadata.

---

### **9. Geospatial Data**

Used in mapping, GPS, remote sensing, etc.

* **Coordinate Cleaning:** Check for valid latitude/longitude ranges.
* **Remove Duplicates:** Based on location ID or proximity.
* **Projection Standardization:** Convert to a consistent CRS (e.g., WGS84).
* **Outlier Detection:** Identify impossible coordinates (e.g., in oceans).
* **Temporal Alignment:** Match time-based location data with time zones.
* **Filter by Region/Bounds:** Crop to regions of interest.

---

### **10. Log Data**

Used in server monitoring, cybersecurity, software logs.

* **Parse Timestamps and Levels:** Extract structured info (e.g., time, severity).
* **Remove Redundant Entries:** Drop duplicate or irrelevant logs.
* **Standardize Formats:** Convert to a consistent structure (JSON, CSV).
* **Mask Sensitive Data:** Remove credentials or PII.
* **Aggregate or Filter Events:** Summarize for analysis.
* **Sessionize Events:** Group log lines by user/session.

---

### **11. Point Cloud Data**

Used in 3D mapping, LiDAR, AR/VR.

* **Remove Noise/Outliers:** Use radius or statistical filters.
* **Downsample:** Use voxel grid to reduce data volume.
* **Align/Normalize:** Rotate/translate point clouds to a common frame.
* **Check for Duplicates:** Eliminate redundant points.
* **Coordinate Correction:** Fix GPS or scanner misalignment.

---

### **12. Genomic/Bioinformatics Data**

Used in DNA sequencing, proteomics, etc.

* **Quality Filtering:** Remove low-quality reads (e.g., based on Phred score).
* **Adapter Trimming:** Remove sequencing adapters.
* **Duplicate Removal:** Eliminate PCR duplicates.
* **Alignment Checks:** Ensure proper mapping to a reference genome.
* **Normalization:** Account for sequencing depth.

---

### **13. Structured Streaming Data**

Like telemetry but includes real-time ingestion (e.g., Kafka, MQTT).

* **Real-time Null Check & Filtering**
* **Latency & Timestamp Corrections**
* **Windowing Events:** Batch events in fixed time windows.
* **Deduplication Across Windows**
* **Schema Evolution Handling**

---



