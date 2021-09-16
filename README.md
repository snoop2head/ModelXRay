# ModelXRay: On-device Machine Learning Model Analyzer and Extractor for Android Apps 

This repository is the static app analysis tool ModelXRay for our paper ["Mind Your Weight(s): A 
Large-scale Study on Insufficient Machine Learning Model Protection in Mobile Apps"(USENIX Security'21)][1].
The main tool is `modelxray.py`, it requires a default config file `modelxray.config`.
The input of ModelXRay can be either an Android APK file or a directory contains APK files, the output is a directory
for each analyzed app. The directory stores the app's profile containing the detailed information about the model files, 
the machine learning libraries and the evidence. For each machine learning model, it also generate one line 
containing the model's meta data and stored in a file called `reports`.

ModelXTractor is based on [Frida](https://frida.re/docs/hacking/). For each app, ModelXRay generates the customized
instrumentation script for Frida. 

## Contents

- `modelxray.py`: our main tool ModelXRay. 
- `modelxray.config`: default config file for ModelXRay. 
- `configure.sh`: check the other tools that ModelXRay relies on. 
- `app_collections.txt`: statistics for our app collection (we do not provide the original app packages which are huge).
- `measure_lic_reuse`: measure the license reuse among different apps.
- `gpu_usage_analysis`: measure how widely GPU acceleration is used among ML apps/libraries.
- `intercept_scripts`: intermediate scripts used by `modelxray.py` to generate analysis script for ModelXTractor.
- `measure_remote_models`: measure how many apps use remote models (or cloud-based ML services).
- `model_encoding_analyzer`: some tools used for analyzing model encoding.
- `ml_app_profile_analysis`: further analysis based on the results generated by ModelXRay on our collected apps.

## Requirements

- ModelXRay is a static analysis tool, it requires a linux/macOS machine with the prerequisite tools listed in `configure.sh`.
- ModelXTractor refers the dynamic analysis, it requires a rooted phone with Frida installed. You also need to install the Android 
  app for analysis. Once this is set, you can run frida on your host machine with the instrumentation script(ModelXTractor) generated
  by ModelXRay. ModelXTractor will store the suspected model buffer on your phone, and you can pull it onto your host machine for
  verification.

## Usage of ModelXRay 

```
usage: modelxray [-h] [-c CONFIG_FILE] [-r] [-l] [-v] [-f] [-s] [-t] [-j] [-p]
                 [-d]    
                 apkpath                                               
                                                                       
positional arguments:                                                  
  apkpath               path to apk file or directory                                                                                          
                                   
optional arguments:    
  -h, --help            show this help message and exit               
  -c CONFIG_FILE, --config-file CONFIG_FILE
                        the path of modelxray config file                                                                                      
  -r, --regenerate-report                                              
                        regenerate report even if report is there
  -l, --log-file        store log in modelxray.log(default to stdout)
  -v, --verbose         verbose logging info                                                                                                   
  -f, --fast-run        run fast by only analyzing library and assets, not
                        smali code
  -s, --space-efficient                                                                                                                        
                        save space by not storing non-machine learning
                        decomposed apps
  -t, --test-only       donot do anything, just test work splitting for 
                        multiprocessing
  -j, --json-script     automatically generate json for dynamic
                        instrumentation java script
  -p, --package-name    use package name as output directory name, default use
                        apk path name
  -d, --decomposed-package
                        start analysis from already decomposed packages 

```



## Issues
If you encounter any problem while using our tool, please open an issue. 

For other communications, you can email sun.zhi [at] northeastern.edu.


## Citing our [paper](https://arxiv.org/pdf/1802.03462.pdf)
```bibtex
@inproceedings {272264,
author = {Zhichuang Sun and Ruimin Sun and Long Lu and Alan Mislove},
title = {Mind Your Weight(s): A Large-scale Study on Insufficient Machine Learning Model Protection in Mobile Apps},
booktitle = {30th {USENIX} Security Symposium ({USENIX} Security 21)},
year = {2021},
url = {https://www.usenix.org/conference/usenixsecurity21/presentation/sun-zhichuang},
publisher = {{USENIX} Association},
month = aug,
}
```

## Disclaimer

All implementations are only research prototypes!

Our code is NOT safe for production use! Please use it only for tests.

## License

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

[http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

[1]: https://www.usenix.org/conference/usenixsecurity21/presentation/sun-zhichuang "Mind Your Weight(s): A Large-scale Study on Insufficient Machine Learning Model Protection in Mobile Apps"

## Q&A

### How to detect encrypted model file?

We use file [entropy](https://en.wikipedia.org/wiki/Entropy_(information_theory)#Data_compression) as indicator for encryption. File entropy in the context of computing measures the degree of randomness of the data. High entropy file fall into three categories:
  * P1: Pure Random Data
  * P2: Compressed files
  * P3: Encrypted files

For suspected model files with high entropy, we rule out P1, assuming it is a model file. Compressed files usually can be detected with file format checking. 
For example, we can use file extention(.zip, .gzip, etc.) to rule out P2. For the rest, we now have some confidence that the high entropy file is encrypted.
See the paper for how we decide on the boundary of encryption for file entropy value. By the way, file entropy can easily calculated with Linux cmdline tool `ent` ([see here](https://wiki.alpinelinux.org/wiki/Entropy_and_randomness0)).
