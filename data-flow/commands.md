
# coollector client 
source env/bin/activate
python3 collector-client.py ~/Projects/priv-wiki/data-flow/example_data.json                

# query 
./query.sh  -b -c config.yaml '165.227.91.185' ~/Projects/priv-wiki/data-flow/query_output.txt