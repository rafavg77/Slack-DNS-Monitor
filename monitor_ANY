#!/bin/bash
DNS="8.8.8.8"
DATE=$(date +%Y-%m-%d-%H-%M-%S)
WDIR=$(pwd)
ODIR=$(echo $WDIR/OUTPUT)
RECORD="ANY"
SLACK_TOKEN=""
SLACK_CHANNEL="#dns_monitor_any"

init_dir(){
	DOMAIN=$1
	if [ -e $ODIR/$DOMAIN/$RECORD ]; then
		echo ""
	else
		mkdir -p $ODIR/$DOMAIN/$RECORD
	fi
}

monitor(){
	DOMAINS=$(cat domains.txt)
	for DOMAIN in $DOMAINS; do
		init_dir $DOMAIN
		sleep 2
		query_dns $DOMAIN
	done
}

query_dns(){
	DOMAIN=$1
	sleep 5
	echo "Query DNS to $DOMAIN"
	dig @$DNS $DOMAIN ANY | sed -n '/QUESTION SECTION/,/Query time/p' | grep -v "QUESTION SECTION" | grep -v "Query time" > $ODIR/$DOMAIN/$RECORD/$DOMAIN-$DATE
	check_diff $DOMAIN
}

check_diff(){
	DOMAIN=$1
	FOLD=$(ls -1tr $ODIR/$DOMAIN/$RECORD | tail -2 | sed -n '1p')
	FNEW=$(ls -1tr $ODIR/$DOMAIN/$RECORD | tail -2 | sed -n '2p')
	DIFF=$(diff $ODIR/$DOMAIN/$RECORD/$FOLD $ODIR/$DOMAIN/$RECORD/$FNEW)

	if [  "$DIFF" ]; then
		echo "Changes"
	else
		echo "No Changes"
	fi

	if [ $FOLD ]; then 
		slack $ODIR/$DOMAIN/$RECORD/$FOLD $DOMAIN "OLD" 
	fi
	sleep 3
	if [ $FNEW ]; then 
		slack $ODIR/$DOMAIN/$RECORD/$FNEW $DOMAIN "NEW" 
	fi
}



slack(){
	FILE=$1
	DOMAIN=$2
	TIME=$3
	curl -s -F file=@$FILE -F initial_comment="DNS Monitor for $DOMAIN $TIME" -F channels=$SLACK_CHANNEL -F token=$SLACK_TOKEN https://slack.com/api/files.upload > /dev/null
}

run(){
	monitor
}

run
