import boto3

def list_ec2_instances_with_unencrypted_volumes(region_name):
    ec2 = boto3.client('ec2', region_name=region_name)

    print(f"\nChecking EC2 instances in region: {region_name}\n")
    paginator = ec2.get_paginator('describe_instances')
    page_iterator = paginator.paginate()

    instances_with_unencrypted_vols = []

    for page in page_iterator:
        for reservation in page['Reservations']:
            for instance in reservation['Instances']:
                instance_id = instance['InstanceId']
                instance_name = None
                for tag in instance.get('Tags', []):
                    if tag['Key'] == 'Name':
                        instance_name = tag['Value']
                        break

                print(f" Instance: {instance_id} ({instance_name or 'No Name'})")

                for mapping in instance.get('BlockDeviceMappings', []):
                    ebs = mapping.get('Ebs')
                    if not ebs:
                        continue

                    volume_id = ebs['VolumeId']
                    volume = ec2.describe_volumes(VolumeIds=[volume_id])['Volumes'][0]
                    encrypted = volume['Encrypted']

                    if not encrypted:
                        print(f" Unencrypted Volume: {volume_id}")
                        instances_with_unencrypted_vols.append(instance_id)
                    else:
                        print(f"  Encrypted Volume: {volume_id}")

    # Summary
    print("\nSummary of Instances with Unencrypted Volumes:")
    if instances_with_unencrypted_vols:
        for inst in set(instances_with_unencrypted_vols):
            print(f"  {inst}")
    else:
        print(" All instances have encrypted volumes.")

if _name_ == "_main_":
    region = "us-east-1"  
    list_ec2_instances_with_unencrypted_volumes(region)
