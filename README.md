---


---

<h2 id="installation-guide-for-geauniversal-04112019">Installation Guide for GEAUniversal (04/11/2019)</h2>
<h3 id="convention">Convention</h3>
<p>General option during installation:</p>
<p><code>--genome-id</code> can always be replaced by <code>--genome-file</code><br>
<code>--file</code> always represent current data input file<br>
<code>--force</code> : skip the confirmation step (answer Y/N)<br>
<code>--override</code>:  delete the existing data before adding. Otherwise it will be adding/updating</p>
<p>current directory:  <code>/opt/geneatlasdb</code></p>
<h3 id="pre-requisites-for-linux-distribution-and-packages">Pre-requisites for Linux distribution and packages</h3>
<p>Ubuntu 16.04</p>
<pre><code>sudo apt install python-pip* python-virtualenv libmysqlclient-dev libapache2-mod-wsgi sqlite3 libxml2-dev graphviz*
</code></pre>
<p>Centos 7, with mysql client: mariadb-5.5.56-2.el7.x86_64</p>
<pre><code>sudo yum -y install epel-release 
sudo yum -y python-pandas python-devel python2-pip python-virtualenv* MySQL-python python-sqlalchemy python-plumbum python2-sh graphviz graphviz-devel 
</code></pre>
<h3 id="install-apprun-server-to-be-completed-skip-this-step">Install APPRUN server (to-be-completed; skip this step!)</h3>
<p>Install java; copy apprun package; set up pin and port; start service by crontab</p>
<h3 id="r-installation">R installation</h3>
<p>Install R(&gt;=3.5.0) and related R packages under Linux</p>
<pre><code>sudo yum install R   #or sudo yum update R
</code></pre>
<p>Enter R interface</p>
<pre><code>  sudo R
</code></pre>
<p>Under R interface:</p>
<pre><code>install.packages("optparse")
if (!requireNamespace("BiocManager", quietly = TRUE))
install.packages("BiocManager")
BiocManager::install("DESeq2", version = "3.8")
BiocManager::install("BiocParallel", version = "3.8")
BiocManager::install("WGCNA", version="3.8")
BiocManager::install("dynamicTreeCut", version="3.8")
BiocManager::install("fastcluster", version="3.8")
quit()
</code></pre>
<h3 id="install-python-flask-and-other-python-packages">Install Python flask and other Python packages</h3>
<p>Suppose installation folder is <code>/opt/geneatlasdb</code>. All future operations will use it as working folder.</p>
<pre><code>mkdir /opt/geneatlasdb
cd /opt/geneatlasdb   ##working folder in all future operations
virtualenv venv
source venv/bin/activate
pip install --upgrade pip

echo "
Click==7.0
Flask==1.0.2
Flask-Session==0.3.1
Jinja2==2.10
MarkupSafe==1.1.0
MySQL-python==1.2.5
SQLAlchemy==1.2.17
Werkzeug==0.14.1
itsdangerous==1.1.0
wsgiref==0.1.2

gff3==0.3.0
numpy==1.11.1
pandas==0.18.1
plumbum==1.6.2
scipy==0.18.0
sh==1.11
Pillow==5.1.0
requests==2.18.4
" &gt; requirements.txt
pip install -r requirements.txt

pip install pygraphviz==1.3.1 --install-option="--include-path=/usr/include/graphviz/" --install-option="--library-path=/usr/lib/graphviz"
pip install git+https://github.com/xinbindai/goenrich.git
</code></pre>
<h3 id="foldersfiles-required-by-geauniversal">Folders/Files required by GEAUniversal</h3>
<p>Assume /opt/geneatlasdb as installation folder.<br>
copy atlasapp and biowebtools directory into /opt/geneatlasdb<br>
copy <a href="http://runserver.py">runserver.py</a> into /opt/geneatlasdb</p>
<h3 id="python-initialization">Python Initialization</h3>
<pre><code>mkdir -p local/py
cp atlasapp/example_local_settings.py  local/py/geneatlas_settings.py
touch local/py/__init__.py
</code></pre>
<h3 id="apprun-client-setting-in-localpygeneatlas_settings">Apprun client setting in local/py/geneatlas_settings</h3>
<pre><code>APPRUN_HOST='localhost'
APPRUN_PORT=
APPRUN_PIN=
APRRUN_LIFESPAN=30
</code></pre>
<h3 id="database-initialization">Database Initialization</h3>
<ul>
<li>
<p>In local/py/geneatlas_settings, set up DBNAME, DBHOSTNAME, DBUSERNAME, DBPASSWORD per your server MySQL settings.</p>
</li>
<li>
<p>Create database and tables in <code>DBNAME</code> database:</p>
<p>Supposed you are under /opt/geneatlasdb:</p>
<pre><code>/atlasapp/geneatlas rdbms --action add
</code></pre>
<p>Optionally, to drop database, try:</p>
<pre><code>atlasapp/geneatlas rdbms --action remove
</code></pre>
</li>
</ul>
<h3 id="load-control-vocabulary-cv-table">Load control vocabulary (CV) table</h3>
<p>Batch load cv information from file:</p>
<pre><code>atlasapp/geneatlas cv --action add --file atlasapp/install/cv.txt
</code></pre>
<p><code>cv.txt</code> is tab-delimited file with name and definition. Only name is mandatory  column.</p>
<p>Or, you can specify individual cv record like this:</p>
<pre><code>atlasapp/geneatlas cv --action add --name KEGG --definition "xxx"
</code></pre>
<h3 id="load-cv-terms-into-defined-class-table">Load CV terms into defined class table</h3>
<pre><code>atlasapp/geneatlas cvterm --action add --classname Relationshiptype --cv-name relationship --file atlasapp/install/relationshiptype.txt --force

atlasapp/geneatlas cvterm --action add --classname Featuretype --cv-name sequence --file atlasapp/install/featuretype.txt --force

atlasapp/geneatlas cvterm --action add --classname Featuretype --cv-name self_definition --file atlasapp/install/self_definition.txt --force

atlasapp/geneatlas cvterm --action add --classname Treatment --cv-name plant_experimental_condition --file atlasapp/install/treatment.txt --force

atlasapp/geneatlas cvterm --action add --classname Anatomy --cv-name plant_ontology --file atlasapp/install/anatomy.txt --force
</code></pre>
<p>In above command, <code>--cv-name</code> is CV name that is one case of first column in <code>cv.txt</code>. <code>--file</code> followe a CV term file, which is a tab-delimited file with acc, name, definition and is_obsolete columns. Only acc is mandatory column.</p>
<p>To remove cvterms from specified class, try <code>--action remove</code> with <code>--classname</code> and <code>--cv-name</code> as below:</p>
<pre><code>atlasapp/geneatlas cvterm --action remove --classname Anatomy --cv-name plant_ontology
</code></pre>
<h3 id="update-apprun-port-hostname-and-port-in-geneatlas_settings.py">Update APPRUN port, hostname and port in geneatlas_settings.py</h3>
<p>to-be-completed</p>
<h3 id="load-genome-meta-information">Load genome meta information</h3>
<p>Go to data folder which hosts your genome/expression data.</p>
<pre><code> atlasapp/geneatlas genome --action add --file genome-meta.tsv
</code></pre>
<p>Example of genome-meta.tsv:</p>
<blockquote>
<p>id  1<br>
species Arabidopsis thaliana<br>
genotype    col<br>
annotation  Araport 11<br>
description Arabidopsis thaliana, JCVI, Araport 11<br>
featuregroup    gene<br>
featurelink <a href="https://apps.araport.org/thalemine/report.do?id=%25s">https://apps.araport.org/thalemine/report.do?id=%s</a><br>
#gene_featuretypes<br>
#transcript_featuretypes<br>
#chromosome_featuretypes<br>
gene_profiledemo    AT5G65720 AT5G65720 AT5G65720 AT1G19940 AT5G49140 AT5G36930 AT4G19050<br>
transcript_profiledemo  AT5G65720.1 AT5G65720.1 AT5G65720.1 AT1G19940.1 AT5G49140.1 AT5G36930.1 AT4G19050.1<br>
chrdemo Chr2 1 200000</p>
</blockquote>
<p># indicate comment line. The first word in each line is keyword which describe the genome. Only id, species and annotation are required.</p>
<p>Here,  genome id can be a NCBI Taxonomy ID for the genome.   However, If you plan to add multiply genome assembly/annotation releases for the same species, you wont choose NCBI taxonomy ID.</p>
<p>To list or remove genome meta data:</p>
<pre><code>atlasapp/geneatlas genome --action list
atlasapp/geneatlas genome --action remove --id 1
</code></pre>
<p>In this case, <code>-id 1</code> is genome ID in genome meta tsv file.</p>
<h3 id="load-feature-genetranscript-data-into-rdbms-database">Load feature (gene/transcript) data into RDBMS database</h3>
<ol>
<li>
<p>load transcript in FASTA format (only transcript will be loaded, no gene information)</p>
<pre><code>atlasapp/geneatlas feature --action add --genome-file genome-meta.tsv --fas transcript.fa
</code></pre>
<p>or, use genome ID to replace genome meta file:</p>
<pre><code>atlasapp/geneatlas feature --action add --genome-id 1 --fas transcript.fa
</code></pre>
</li>
<li>
<p>or, load features(chr/contig/gene/transcript) from GFF file<br>
<strong>simply case:</strong></p>
<pre><code>atlasapp/geneatlas feature --action add --genome-file genome-meta.tsv --gff genome.gff
</code></pre>
<p><strong>complicate case:</strong><br>
for gff downloaded from NCBI database (protein name in CDS line, locus name in Name property for gene/transcript):</p>
<pre><code>atlasapp/geneatlas feature --action add --genome-file genome-meta.tsv --gff genome.gff --subelements --makeprotein --desckeys product Note --acckeys Name ID
</code></pre>
</li>
<li>
<p>remove all features(optional)</p>
<pre><code>atlasapp/geneatlas feature --action remove --genome-file genome-meta.tsv
</code></pre>
</li>
</ol>
<h3 id="load-rna-seq-sample-metadata-by-experiment">Load RNA-seq sample metadata by experiment</h3>
<ul>
<li>
<p>Load sample metadata in term of experiment:</p>
<pre><code>atlasapp/geneatlas experiment --action add --genome-file genome-meta.tsv --file rnaseq-meta.csv --force    
</code></pre>
<p><code>--file</code> meta data file name for experiment/condition/sample structure<br>
<code>--fileformat</code>  csv or tsv.  csv is default format!</p>
</li>
<li>
<p>Optional, list all exp/condition/replicate metadata in csv/tsv format</p>
<pre><code>atlasapp/geneatlas experiment --action list --genome-file genome-meta.tsv
</code></pre>
<p><code>--fileformat</code> csv or tsv format</p>
</li>
<li>
<p>Optional, remove all experiments (including their conditions, samples, data and cache) for a species:</p>
<pre><code>atlasapp/geneatlas experiment --action remove --genome-file genome-meta.tsv
</code></pre>
</li>
<li>
<p>Optional, remove an experiment (including its conditions, samples and expression data) by interactive mode (list exps and ask user to choose)</p>
<pre><code>atlasapp/geneatlas experiment --action remove --genome-file genome-meta.tsv --interactive
</code></pre>
<p>User will be asked for the exp ID to be deleted .</p>
</li>
</ul>
<h3 id="load-expression-data-from-rsem-or-salmon-for-samples">Load expression data from RSEM or Salmon for samples:</h3>
<pre><code>atlasapp/geneatlas expression --action add --genome-file genome-meta.tsv --file rnaseq-meta.csv --path RSEMSTAR__Araport11 --up rsem --featuregroup both --force
</code></pre>
<p><code>--featuregroup</code>: gene, transcript or both<br>
<code>--fileformat</code>: meta data file format, csv or tsv<br>
<code>--force</code>: directly go ahead without Y/N confirmation</p>
<h3 id="install-go-annotation-optional">Install GO annotation (optional)</h3>
<p>Below is an example .</p>
<pre><code>atlasapp/geneatlas go --action add --genome-file genome-meta.tsv --obofile go-basic.obo --file gene transcript-GO.tsv --fileformat FEATURE2GO  --desc 'Electronic annotation by BLAST against plant uniprot database 201807'
</code></pre>
<p><code>--file</code> is a GO annotation file. If <code>--fileformat</code> is <code>FEATURE2GO</code>, the first column in the GO annotation file should be gene or transcript acc and the second column should be GO term acc (as described in obofile).</p>
<h3 id="install-kegg-annotation-optional">Install KEGG annotation (optional)</h3>
<p><code>--file</code> is a tab-delimited text file with two columns, the first column should be gene or transcript acc.</p>
<pre><code>atlasapp/geneatlas kegg --action add --genome-file genome-meta.tsv --file gene-KO.tsv --taxonomy Plants
</code></pre>
<h3 id="update-genetranscript-description-optional">Update gene/transcript description (optional)</h3>
<ul>
<li>
<p>Delete existing gene description in database and add new description from <code>--file</code>.</p>
<pre><code>atlasapp/geneatlas  feature-desc --action add --genome-file genome-meta.tsv --override --file gene-desc.tsv --featuregroup gene
</code></pre>
<p>By simialr way, the following command will delete old description and add new description for transcript:</p>
<pre><code>atlasapp/geneatlas  feature-desc --action add --genome-file genome-meta.tsv --override --file transcript-desc.tsv --featuregroup transcript
</code></pre>
<p>In above two examples,  <code>--featuregroup</code> is <code>gene</code> or <code>transcript</code>,  the first column in <code>--file</code> should match it.</p>
<p><code>--file</code> specify a description tsv file, in which the first column is gene/transcript acc and the second column is description.</p>
<p>If <code>--featuregroup</code> is <code>both</code>, then it will update both gene and transcript using the information in <code>--file</code> (either gene or transcript). See this example:</p>
<pre><code>atlasapp/geneatlas  feature-desc --action add --genome-file genome-meta.tsv --override --file transcript-desc.tsv --featuregroup both
</code></pre>
<p>Both gene and transcript will be updated by above command.</p>
</li>
<li>
<p>Append GO/KEGG annotation into feature description.</p>
<pre><code>atlasapp/geneatlas  feature-desc --action add --genome-file genome-meta.tsv --cvnames biological_process cellular_component molecular_function KEGG  --featuregroup both --includepropname
</code></pre>
<p><code>--includepropname</code>: will include the name column in prop table. Otherwise, only acc column will be included.</p>
</li>
</ul>
<h3 id="add-feature-prop-optional">Add feature prop (optional)</h3>
<p>Firstly, register CV name for this kind of prop:</p>
<pre><code>atlasapp/geneatlas cv --action add --name HOMOLOG --definition 'ortholog or paralog group'
</code></pre>
<p>Then, add terms into Prop database table for above cv:</p>
<pre><code>atlasapp/geneatlas cvterm --action add --classname Prop --cv-name HOMOLOG --file homolog_ontology.txt
</code></pre>
<p>In above example, <code>homolog_ontology.txt</code> is a pre-defined CV term file, in which the first column is unique acc.</p>
<p>Next, add feature_prop into database</p>
<pre><code>/opt/geneatlasdb/atlasapp/geneatlas feature-prop --action add --cv-name HOMOLOG --genome-file genome-meta.tsv --file gene_homolog.txt --extend-feature
</code></pre>
<p>In above example, <code>gene_homolog.txt</code> include two columns (gene/transcript acc and CV term acc)</p>
<h3 id="test-installation-on-developing-mode">Test installation on developing mode</h3>
<p>Make sure both session and cache directories have 1777 mode.</p>
<p>Next,  type:</p>
<pre><code>cd /opt/geneatlasdb
source venv/bin/activate
python runserver.py
</code></pre>
<p>Then, open browser, type <code>http://localhost:5000/</code></p>
<h3 id="deploy-on-product-server">Deploy on product server</h3>
<p><strong>Deploy the installation on Apache 2.4</strong></p>
<p>Suppose installation folder is <code>/opt/geneatlasdb</code> and URI is <code>/geneatlasdb</code>.</p>
<pre><code>echo "
WSGIScriptAlias /geneatlasdb  /opt/geneatlasdb/runserver.py
&lt;Directory /opt/geneatlasdb&gt;
WSGIScriptReloading On
Require all granted
#Order allow,deny
#Allow from all
&lt;/Directory&gt;
" &gt; /tmp/geneatlasdb.conf
</code></pre>
<p>Then, for CentOS 7:</p>
<pre><code>sudo mv /tmp/geneatlasdb.conf /etc/httpd/conf.d/
sudo systemctl restart httpd
</code></pre>
<p>**Others: **</p>
<p>list or remove cv:</p>
<pre><code>atlasapp/geneatlasOthers usage:-cmeO ge i emo    
tnata cv --action list

atlasapp/geneatlas cv --action remove --cvname HOMOLOG
</code></pre>
<p>list or remove props by cv_name:</p>
<pre><code>atlasapp/geneatlaspnata  --action list --cv-name HOMOLOG

atlasapp/geneatlas prop --action remove --cv-_name HOMOLOG
</code></pre>
<h3 id="convention-in-geauniversal">Convention in GEAUniversal</h3>
<h4 id="references-in-experiment-table">References in Experiment table</h4>
<p>Pubmed ID separated with space, example:</p>
<blockquote>
<p>PMID:123456  PMID:34343434</p>
</blockquote>
<h3 id="misc">Misc</h3>
<h4 id="download-transcript-protein-and-gene-transcript-relationship">Download transcript-protein and gene-transcript relationship</h4>
<p>Go to species page and click download icon</p>

