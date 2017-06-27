# Exploring Network Programmability with Python and YANG

## Intro

Welcome to the lab, this guide will help you get started programming network devices using YANG and Python over gRPC. Using YANG and Python, the network is easier to program and configuration is more predictable. 

## Get the environment set up.

1. Download the Github 

   ```git clone http://www.github.com/skkumaravel/devnet-1229```

2. cd into directory and start the vagrant box.

   ```cd devnet-1229 && vagrant up```

3. Install python gRPC library

   ```pip install iosxr_grpc``` 

## Create your first gRPC call. 

1. Open up a new file called `example.py` in your favorite editor and add the following code.

```python
from iosxr_grpc.cisco_grpc_client import CiscoGRPCClient
from grpc.framework.interfaces.face.face import AbortionError


def main():
    client = CiscoGRPCClient('127.0.0.1', 57777, 10, 'vagrant', 'vagrant')
    path = ''
    try:
        err, result = client.getconfig(path)
        if err:
            print(err)
        print(result)
    except AbortionError:
        print('Unable to connect to local box, check your gRPC destination.')


if __name__ == '__main__':
    main()
```

2. Run it.

   ```python example.py```

You should get the config of the entire box in yang format.

## Changing an interface

1. Test basic connectivity

   1. `vagrant ssh devbox`
   2. `ping 11.1.1.10`
   3. `exit`

2. Change the path to the interface YANG path in `example.py`.

   `path = '{"openconfig-interfaces:interfaces": {}}'`

3. Lets run the output to a file and look at it.

   `python example.py > interface.json`

4. Lets turn off a interface. Edit `interface.json` and change gig 0/0/0/0 enabled to false.

   ```json
       "name": "GigabitEthernet0/0/0/0",
       "config": {
        "name": "GigabitEthernet0/0/0/0",
        "type": "iana-if-type:ethernetCsmacd",
        "enabled": false
       },
   ```

5. Create `merge.py` and add the following code. This will allow you to open a file and merge the config in.

   ```python
   from iosxr_grpc.cisco_grpc_client import CiscoGRPCClient
   from grpc.framework.interfaces.face.face import AbortionError


   def main2():
       client = CiscoGRPCClient('127.0.0.1', 57777, 10, 'vagrant', 'vagrant')
       path = open('interface.json').read()
       try:
           err = client.mergeconfig(path)
           if err:
               print(err)
       except AbortionError:
           print('Unable to connect to local box, check your gRPC destination.')

   if __name__ == '__main__':
       main2()
   ```

6. Run the program to merge the config.

   `python merge.py`

7. Test connectivity again.

   1. `vagrant ssh devbox`
   2. `ping 11.1.1.10`
   3. `exit`

## Conclusion

There are a lot more configuration options available in the library that can be used with automation. 

Check out:

- [gRPC Python Library](https://github.com/cisco-grpc-connection-libs/ios-xr-grpc-python)
- [YANG models](https://github.com/YangModels/yang)
- [XR docs & tutorials](https://xrdocs.github.io)
