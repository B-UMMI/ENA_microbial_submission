# Submission of microbial sample data to the European Nucleotide Archive (ENA)

We suggest that you start by reading the [General Guide On ENA Data Submission](https://ena-docs.readthedocs.io/en/latest/submit/general-guide.html). This repo contains scripts and instructions to submit raw reads programmatically. The only step that is performed interactively is the [Study Registration](https://ena-docs.readthedocs.io/en/latest/submit/study.html). You need to have a Webin submission account to be able to submit data to the ENA (information about creating an account [here](https://ena-docs.readthedocs.io/en/latest/submit/general-guide/registration.html#register-a-submission-account)).

## Register Study

Start by going through the [Register a Study Interactively](https://ena-docs.readthedocs.io/en/latest/submit/study/interactive.html). We find it easier to register a new Study interactively, especially when the raw reads we want to submit will become associated with a single Study, but you can find instructions to do this programmatically on the [Register a Study Programmatically](https://ena-docs.readthedocs.io/en/latest/submit/study/programmatic.html) section. We suggest that you define a release date well beyond the present date to avoid making the study publicly available when you still need to edit it. Do not forget to download the file with the BioProject and the Study accession numbers.

## Register Samples

The next step is to register your samples. You can find information about this step on the [How to Register Samples](https://ena-docs.readthedocs.io/en/latest/submit/samples.html) and [Register Samples Programmatically](https://ena-docs.readthedocs.io/en/latest/submit/samples/programmatic.html) sections. We will use the ERC000028 checklist (ENA prokaryotic pathogen minimal sample checklist, available [here](https://www.ebi.ac.uk/ena/browser/view/ERC000028)). We will be constructing the XML based on the fields in the checklist (we will not modify the checklist file for submission). This repo includes a TSV file, `test_sample_metadata.tsv`, with example metadata necessary to create the XML file used to register samples in the ENA. You can leave the `center_name` field empty. The center name is automatically assigned from submission account details and any value that you include in the metadata will be ignored (more information about this [here](https://ena-docs.readthedocs.io/en/latest/submit/general-guide/programmatic.html#identifying-submitters). To create the XML file based on your sample metadata, run the `ena_samples_xml.py` script. Adapt the following command:

```
python ena_samples_xml.py -i test_sample_metadata.tsv -o sample.xml -c ERC000028
```

This will create an XML file with the data to register the samples and the accompanying submission XML file, `submission.xml`.
To submit the XML files and register your Samples, you can run the following command:

```
curl -u username:password -F "SUBMISSION=@submission.xml" -F "SAMPLE=@sample.xml" "https://www.ebi.ac.uk/ena/submit/drop-box/submit/"
```

Save the response to a file.
If you get `curl: (26) Failed to open/read local data from file/application`, the name of the submission or samples XML might be incorrect. Check that and try again. More information is available [here](https://ena-docs.readthedocs.io/en/latest/submit/samples/programmatic.html#submit-the-xmls-using-curl). Please provide your Webin submission account credentials using the `username` and `password` (do not add single or double quotes to the username or password). You should test your submissions before submitting the data to the production service. Find more information about using the Test service in the [Test and production services](https://ena-docs.readthedocs.io/en/latest/submit/samples/programmatic.html#test-and-production-services) section. If you need to update the metadata for samples that have already been registered, simply update the data in the XML file used to register the samples and change the ADD tag in submission XML to MODIFY (example [here](https://ena-docs.readthedocs.io/en/latest/update/metadata/programmatic-sample.html).

## Prepare and Upload Read Files

The FASTQ files must be compressed using gzip or bzip2 (check the section about [Preparing A File For Upload](https://ena-docs.readthedocs.io/en/latest/submit/fileprep/preparation.html#preparing-a-file-for-upload)). The `ena_reads.xml.py` script determines the checksums and creates the XML file to register your Runs and Experiments. In this step you only need to make sure that your files are compressed and that the filenames are in the format `<SAMPLE_ID>_1.fastq.gz` (The `SAMPLE_ID` is the value used as `alias` in the TSV file with the sample metadata).
To upload read files:

1. Open a terminal and type `lftp webin2.ebi.ac.uk -u Webin-xxxxx`, filling in your Webin username (must have `lftp` installed).
2. Enter your password when prompted.
3. Type `ls` to check the content of your drop box.
4. Use the `mput <filename>` command to upload files.
5. Use the `bye` command to exit the ftp client.

This will upload the read files to your private Webin file upload area using FTP. After you register the Runs and Experiments, the server will link the files to the Run and Experiment records automatically based on values passed for each Run and Experiment.

## Register Runs and Experiments

You need to create a TSV file with the data necessary to register the Runs and Experiments. This repo includes a TSV file, `test_run_experiment_metadata.tsv`, with example metadata necessary to create the XML file used to register Runs and Experiments in the ENA. To create the XML file to register your Runs and Experiments you should run the `ena_reads.xml.py` script. Adapt the following command:

The `studyID` is the BioProject accession.

```
python ena_reads.xml.py -i test_run_experiment_metadata.tsv -o datasetID --study studyID --reads readsDir
```

To submit the XML files and register your Runs and Experiments, you can run the following command:

```
curl -u username:password -F "SUBMISSION=@submission.xml" -F "EXPERIMENT=@experiment.xml" -F "RUN=@run.xml" "https://www.ebi.ac.uk/ena/submit/drop-box/submit/"
```
