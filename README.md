# ami created using for loop if there are multiple instance_id's
#!/bin/bash
region=us-west-1
topic_arn=arn:aws:sns:us-west-1:058264248093:930pm-demo
aws ec2 describe-instances --filters "Name=tag:created_by,Values=sai" --region $region | grep -i instanceid | cut -d ":" -f 2 | cut -d "," -f 1 | tr -d '"' > instance_id.txt
if [ ! -s instance_id.txt ];
 then
    echo "There are no matching records with the given tags please check......"
else
  while IFS= read -r instance_id; do   
    echo "Creating AMI for $instance_id....."
    timestamp=$(date +%Y%m%d%H%M%S)
    ami_name="demo-instance-AMI-$timestamp"
    aws ec2 create-image --instance-id "$instance_id" --name "$ami_name" --description "An AMI for my demo server" && aws sns publish --topic-arn $topic_arn --message "Your ami is created successfully!"
    echo "AMI for $instance_id is created successfully........"
  done < instance_id.txt
 fi

# script to take snapshots

 #!/bin/bash
aws ec2 describe-instances --filters "Name=tag:created_by,Values=sai" --region us-west-1 --query 'Reservations[].Instances[].BlockDeviceMappings[].Ebs[].VolumeId' --output text > instance_id.txt
while IFS= read -r volume_id; do   
    echo "Creating snapshot for: $volume_id....."
    aws ec2 create-snapshot --volume-id "$volume_id" --description "Snapshot for volume $volume_id" && aws sns publish --topic-arn arn:aws:sns:us-west-1:058264248093:930pm-demo --message "Your snapshot is created successfully!"
 done < instance_id.txt