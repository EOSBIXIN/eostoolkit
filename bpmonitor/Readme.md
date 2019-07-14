# bpmonitor.py 
### Functions:
- Monitor the rank of bp account and send notification message
- Send warning message when not create blocks in 21th
- Calculate the rewards everyday and save into ```reward_output``` file.

### Implementation:
In the config file, we can supply several API endpoints to query the infomation from the EOS mainnet. Several API endpoints can avoid single point of failure and deferred blocks synchronization. Every endpoint will create a thread and check with mainnet periodically. 

The rewards calculate by the average vote rate from the last 24 hours. 

The functions have been realized clearly, what you need to do is just rewrite your notification fucntion in ```send_warning()```. The default notification function is using AliSMS service. 

The ```bpmonitor.json``` is easy to be modified and you can add other items by yourself.

### bpmonitor.json 
```
notify_interval: Same warning message will not be send in the interval. 
http_urls: the API endpoints list
bpaccount: the watched bp account 
reward_output: the file to save daily rewards
```