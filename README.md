# Repository for AMIA 2024 Poster Submission: "Identifying Adverse Drug Events Using Ensemble Models"

# Requirements:
* Directory structure: all files should be in the same directory, at the same level.
* Requirements available in requirements.txt. Run pip install -r requirements.txt.
* Created with Python 3.9.9

# Instructions: 
1. Create a folder with BRAT annotated notes from the MIMIC 3 corpus.
2. Pass that folder path into BRAT_Parser.
3. Once that module processes the files, run the SVM / T5 / BERT models on the files to predict.
4. Run the error analysis file to extract relevant errors. 

# Details:
All models were trained on NVIDIA A100s via the GMU Office of Research Computing.
* SVM sigmoid kernel: trained using C = 10, class weight of noADE: 1, ADE: 5

* Bio_Clinical BERT: batch size = 8, trained for 5 epochs, learning rate = 5e-5, AdamW optimization. All other hyperparameters are the HF/PT defaults.
  
* T5-large: batch size = 8, trained for 5 epochs, learning rate = 5e-5, AdamW optimization. All other hyperparameters are the HF/PT defaults.

# Pipeline: 
Given folders of certain note types (e.g. "general", "consult", etc.), each folder contains BRAT annotated data (i.e. raw .txt file of EHR, .ann file of human-labeled BRAT data). Each .ann file contains a wealth of information, including an ADE relation between a Drug and Problem event, and their respective character spans in the txt file. 

The BRAT_Parser module runs on these file pairs and breaks the .txt file into sentences via the sci_spacy_web_md tokenizer. Then, those sentences are labeled as ADE (class 1) if they contain the "Problem" event span, otherwise they are labeled as noADE (class 0). The parser outputs this data as dictionaries of key: value pairs for each note type. 

The SVM / BERT / T5 modules receive that data as input. They tokenize with their respective tokenizers (using the AutoTokenizer class from HF for T5/BERT, the Count Vectorizer from Scikit for SVM). Then, the models are trained with the appropriate training loops and prior hyperparameters, and tested on the test set. The validation set was used prior to this step. 

The Ensemble models, present in the Error_Analaysis file, receive the predictions of the 3 prior models. They order the predictions, based on their original sentence index, and perform the correct operation: the union model predicts class 1 if any of the models predict class 1, the majority model predicts the same as the majority of models. 

Error analysis is performed in several aspects: erroroneously classified sentences are examined based on their length in characters, the types of BRAT data they contain (i.e. only Problem events, or both Drug and Problem events), the original note type they came from, etc. 
