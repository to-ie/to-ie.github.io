---
layout: post
title: "Running Athena queries from terminal" 
categories: AWS
date : 2023-06-26 15:14:50 
---

I recently had an (oddly-specific) assignement, at work, where I needed to run a bunch of AWS Athena queries from the terminal. If this is what you're after, keep reading. 

## The queries

First, create a folder that will hold the various queries you need to run. I am the creative type, so I called this folder `queries`.

In there, I store the queries I need to run, one per file. For example, the file `firstquery` holds the following: `SELECT * FROM name_of_my_athena_table;`

So I end up with the following structure:

```
.
├── queries
│   ├── firstquery
│   ├── secondquery
│   └── thirdquery
```

## The script

We then need to create the script that will read the queries. I created a file called `run_athena_query.sh` with the following code: 

```
#!/bin/bash
while IFS='' read -r sql || [[ -n "$sql" ]]; do
    aws athena start-query-execution \
    --query-string "$sql" \
    --query-execution-context Database=<name-of-database> \
    --result-configuration OutputLocation=s3://<bucket-location-where-results-are-stored> \
    --output text
done < "$1"
```
(Make sure to replace `name-of-database` and `<bucket-location-where-results-are-stored>` with your own bits!)

You now should have something that looks like this: 
```
.
├── queries
│   ├── firstquery
│   ├── secondquery
│   └── thirdquery
└── run_athena_query.sh

```


## Running the query

Obviously, you need to have configured your [AWS CLI](https://aws.amazon.com/cli/) accordingly (or if you're like me and have multiple accounts with roles to be assumed, check out [Awsume](https://blog.t-o.ie/aws/2022/11/04/Awsume/)). 

Then, simply run:
```
bash run_athena_query.sh queries/firstquery
```

The script will spit out the name of the file that stores the results. Your terminal should look something like this: 

```
$ bash run_athena_query.sh queries/firstquery
a2b95641-a045-46df-8376-17a5d3acc2c2
```

You can then fish out this file from the bucket defined in `run_athena_query.sh` (remember `<bucket-location-where-results-are-stored>`?)

That's all. Easy right?