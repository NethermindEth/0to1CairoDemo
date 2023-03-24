## Steps to upgrade from cairo 0 upgradeable contract to cairo 1 upgradeable contract
This method uses replace_class_syscall to remove the proxy pattern altogether. At the end of migration the contract utlizes an implementation class hash wich has an upgrade method in it. This assumes replace_call_syscall will not be deprecated and the Proxy pattern in cairo0 will not be the dominant pattern for upgradeable contracts in cairo1.

Install the latest v0.11.0 cairo-lang package.

```shell
cd cairo0
protostar build
starknet declare --contract build/proxy.json --account version_11_2
```
class hash 0xeafb0413e759430def79539db681f8a4eb98cf4196fe457077d694c6aeeb82
transaction hash for declaration above 0x6d72968648f4feab7776ac644137f7008b9b113980169c99fe90a3656852b54


If for some reason declare below doesnt work from CLI, try argentx wallet's developer setting

```shell
starknet declare --contract build/cairo0resolver.json --account version_11_2
```

class hash 0x6cf18bc8305c6b2ce6d1954cf9a502fbd8fe4aac467bd89fb0682de4c077e6c
transaction hash for declaration above 0xe23e6ba05b95cc4e1836b08a64f107d16c2966f9b9343442dc33cf08410d70



selector for 'initializer'  0x2dd76e7ad84dbed81c314ffe5e7a7cacfb8f4836f01af4e913f275f89a3de1a
proxy declare at 0xeafb0413e759430def79539db681f8a4eb98cf4196fe457077d694c6aeeb82
UDC address 0x041a78e741e5af2fec34b695679bc6891742439f7afb8484ecd7766661ad02bf
implementation class hash for cairo0resolver 0x6cf18bc8305c6b2ce6d1954cf9a502fbd8fe4aac467bd89fb0682de4c077e6c
working account address 0x1c15b60bc80306354cc68d83c1fc487b4b0b0e1540fe62c1df9aa3ae4ba396a

```shell
starknet invoke \
    --address 0x041a78e741e5af2fec34b695679bc6891742439f7afb8484ecd7766661ad02bf \
    --abi ../UDC.json \
    --function deployContract \
    --inputs 0xeafb0413e759430def79539db681f8a4eb98cf4196fe457077d694c6aeeb82 0 0 4 0x6cf18bc8305c6b2ce6d1954cf9a502fbd8fe4aac467bd89fb0682de4c077e6c 0x2dd76e7ad84dbed81c314ffe5e7a7cacfb8f4836f01af4e913f275f89a3de1a 1 0x1c15b60bc80306354cc68d83c1fc487b4b0b0e1540fe62c1df9aa3ae4ba396a \
	--account version_11_2
```

transaction hash for above transaction 0x33017a6d35867256b06b34d4f4604e577dbe85c403b036617e13b1655c2c51
contract deployed at 0x000684167f946c40e4fe022e8af0bd6d17a18b04d0a0bed7826fecb891648222

Just go to voyager/starkscan and update/write into the contract (or you can invoke transaction below) to update the contract to have some state
the transaction below updates the domain node 10 to return a starknet_id of 2024 

```shell
starknet invoke \
    --address 0x000684167f946c40e4fe022e8af0bd6d17a18b04d0a0bed7826fecb891648222 \
    --abi ./build/cairo0resolver_abi.json \
    --function set_starknet_id \
    --inputs 10 2024 \
	--account version_11_2
```	

transaction hash for above transaction 0x3c490dc2a84ab9259b0c732b663b51106d31e868580681008b1a6d7749ad457

compile cairo1/cairo1resolver.cairo via cairo1 compiler and please add the steps for compiling etc here
class hash after declaration for cairo1/cairo1resolver.cairo 0x1d7ec1927b0bab67fdaea523906077c6f141eb68ca0e7b8c6403705d67fb1cb

```shell
cd ../cairo1
starknet declare --contract ./cairo1resolver.json --account version_11_2
```


this is to upgrade to cairo1 implementation

```shell
starknet invoke \
    --address 0x000684167f946c40e4fe022e8af0bd6d17a18b04d0a0bed7826fecb891648222 \
    --abi ../cairo0/build/cairo0resolver_abi.json \
    --function upgrade \
    --inputs 0x1d7ec1927b0bab67fdaea523906077c6f141eb68ca0e7b8c6403705d67fb1cb \
        --account version_11_2
```		
		
transaction hash 0x31ee06c302cc24719350ae248d31610478d6b0831804cd5bb27cec21fec16a3

After success completion make sure the state is valid, check voyager to see if the implementation has been upadted and state is in tact. like domain_node 10 resolves to 2024


now we utlize the replace_class_syscall to remove cairo0 proxy altogether.
I could not make the transaction below to work because I didnt have abi and invoke was complaining about type info, may be cairo1 compiler does not output the type just yet? So I just utilized the voyager/starkscan to invoke the upgrade method directly and that worked

```shell
starknet invoke \
    --address 0x000684167f946c40e4fe022e8af0bd6d17a18b04d0a0bed7826fecb891648222 \
    --abi ./cairo1resolver.json \
    --function upgrade \
    --inputs 0x1d7ec1927b0bab67fdaea523906077c6f141eb68ca0e7b8c6403705d67fb1cb \
        --account version_11_2
```
		
after the above transaction is completed the contract at 0x000684167f946c40e4fe022e8af0bd6d17a18b04d0a0bed7826fecb891648222 has been changed from a proxy/implementation type to regular contract with an implementation class of 0x1d7ec1927b0bab67fdaea523906077c6f141eb68ca0e7b8c6403705d67fb1cb

transaction hash for above transaction 0x2d61b7f4b21d0ece5ba4f3abaa87d730112a7b33e19df2ec06e12b77b4db284
