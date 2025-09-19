#### 项目架构
```
GRBAfterglowProcessor/
├── src/
│   ├── __init__.py
│   ├── data_preparation/
│   │   ├── __init__.py
│   │   ├── optical_data_processor.py
│   │   └── xray_data_processor.py
│   ├── temporal_analysis/
│   │   ├── __init__.py
│   │   ├── broken_powerlaw_fitter.py
│   │   └── afterglow_model_fitter.py
│   ├── utils/
│   │   ├── __init__.py
│   │   ├── time_utils.py
│   │   ├── flux_utils.py
│   │   └── file_utils.py
│   ├── config/
│   │   ├── __init__.py
│   │   └── config_manager.py
│   └── main.py
├── tests/
│   ├── test_optical_data.py
│   ├── test_xray_data.py
│   ├── test_fitting.py
│   └── test_utils.py
├── examples/
│   ├── sample_optical_data.csv
│   ├── sample_xray_data.fits
│   └── run_example.py
├── requirements.txt
└── README.md
```
