We did some internal research on our side to evaluate if there are other alternatives for testing batches of CSVs other than a Jupyter or Google Colab notebook. We found some code on our site that may be helpful, but it will require some Python knowledge within your team. 

We attached a file: lakera_automated_results.py. It is a slight modification of the eval script located in our documentation. We made a small adjustment to enable it to process multiple files

We assume your team has several files since you asked about batch loading. You will need to complete a few things before your team runs the script.

If your CSVs are in an acceptable format, we should be able to process multiple CSVs if they are all located in the same testing folder as the attached script on your machine. Your developer can use our recommended script as shown in the docs if that works for you.
Create a folder for testing in your environment
Download script. Move the lakera_automated_results.py script into your testing folder
Move or copy test prompt files (.csv) to that testing folder
Create and activate a virtual Python environment
Download dependencies to your virtual Python environment (imports at top of script file)
You may see errors for each missing dependency when you run the script
Set an environment variable named LAKERA_GUARD_API_KEY in your current shell session and add your API key to it
export LAKERA_GUARD_API_KEY=your_actual_api_key_here
Run script by entering the commands below on your command line
You may need to use python3 command instead of python based on your version
To process all CSVs in testing folder
python lakera_automated_results.py --datasets "*.csv"
To process specific files
python lakera_automated_results.py --datasets file1.csv file2.csv
Feel free to process the scripts one at a time. That way you have control over the size of file processed and time it will take to process them.

Let us know if you have any questions.


#isi comment

python3 lakera_automated_results.py --datasets jailbreak_dataset_full.csv -e dd205ddd924bcea59fd0f48a1d163b575522a5fc69f28d9861fee3023d8e7d6d
