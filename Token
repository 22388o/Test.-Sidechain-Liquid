 1#!/bin/bash
  2set -x
  3
  4shopt -s expand_aliases
  5
  6# ASSUMES elementsd IS ALREADY RUNNING
  7
  8######################################################
  9#                                                    #
 10#    SCRIPT CONFIG - PLEASE REVIEW BEFORE RUNNING    #
 11#                                                    #
 12######################################################
 13
 14# Amend the following:
 15NAME="your asset name here"
 16TICKER="your ticker"
 17# Do not use a domain prefix in the following:
 18DOMAIN="domain.here"
 19# Issue 100 assets using the satoshi unit, dependant on PRECISION when viewed from
 20# applications using Asset Registry data.
 21ASSET_AMOUNT=0.00000100
 22# Issue 1 reissuance token using the satoshi unit, unaffected by PRECISION.
 23TOKEN_AMOUNT=0.00000001
 24
 25# Amend the following if needed:
 26PRECISION=0
 27
 28# Don't change the following:
 29VERSION=0
 30
 31# Change the following to point to your elements-cli binary and liquid live data directory (default is .elements).
 32alias e1-cli="elements-cli -datadir=$HOME/.elements"
 33
 34##############################
 35#                            #
 36#    END OF SCRIPT CONFIG    #
 37#                            #
 38##############################
 39
 40# Exit on error
 41set -o errexit
 42
 43# We will be using the issueasset command and the contract_hash argument:
 44# issueasset <assetamount> <tokenamount> <blind> <contract_hash>
 45
 46NAME="your asset name here"
 47TICKER="ticker here"
 48DOMAIN="domain.here"
 49ASSET_AMOUNT=100
 50TOKEN_AMOUNT=1
 51PRECISION=0
 52
 53# Don't change the following:
 54VERSION=0
 55
 56# As we need to sign the deletion request message later we need
 57# a legacy address. If you prefer to generate a pubkey and sign
 58# outside of Elements you can use a regular address instead.
 59NEWADDR=$(e1-cli getnewaddress "" legacy)
 60
 61VALIDATEADDR=$(e1-cli getaddressinfo $NEWADDR)
 62
 63PUBKEY=$(echo $VALIDATEADDR | jq -r '.pubkey')
 64
 65ASSET_ADDR=$NEWADDR
 66
 67NEWADDR=$(e1-cli getnewaddress "" legacy)
 68
 69TOKEN_ADDR=$NEWADDR
 70
 71# Create the contract and calculate the contract hash
 72# The contract is formatted for use in the Blockstream Asset Registry:
 73
 74CONTRACT='{"entity":{"domain":"'$DOMAIN'"},"issuer_pubkey":"'$PUBKEY'","name":"'$NAME'","precision":'$PRECISION',"ticker":"'$TICKER'","version":'$VERSION'}'
 75
 76# We will hash using openssl, other options are available
 77CONTRACT_HASH=$(echo -n $CONTRACT | openssl dgst -sha256)
 78CONTRACT_HASH=$(echo ${CONTRACT_HASH#"(stdin)= "})
 79
 80# Reverse the hash
 81TEMP=$CONTRACT_HASH
 82LEN=${#TEMP}
 83until [ $LEN -eq "0" ]; do
 84    END=${TEMP:(-2)}
 85    CONTRACT_HASH_REV="$CONTRACT_HASH_REV$END"
 86    TEMP=${TEMP::-2}
 87    LEN=$((LEN-2))
 88done
 89
 90# Issue the asset and pass in the contract hash
 91IA=$(e1-cli issueasset $ASSET_AMOUNT $TOKEN_AMOUNT false $CONTRACT_HASH_REV)
 92
 93# Details of the issuance...
 94ASSET=$(echo $IA | jq -r '.asset')
 95TOKEN=$(echo $IA | jq -r '.token')
 96ISSUETX=$(echo $IA | jq -r '.txid')
 97
 98#####################################
 99#                                   #
100#    ASSET REGISTRY FILE OUTPUTS    #
101#                                   #
102#####################################
103
104# Output the proof file - you need to place this on your domain.
105echo "Authorize linking the domain name $DOMAIN to the Liquid asset $ASSET" > liquid-asset-proof-$ASSET
106
107# Create the bash script to run after you have placed the proof file on your domain
108# that will call the registry and request the asset is registered.
109echo "curl https://assets.blockstream.info/ --data-raw '{\"asset_id\":\"$ASSET\",\"contract\":$CONTRACT}'" > register_asset_$ASSET.sh
110
111# Create the bash script to delete the asset from the registry (if needed later)
112PRIV=$(e1-cli dumpprivkey $ASSET_ADDR)
113SIGNED=$(e1-cli signmessagewithprivkey $PRIV "remove $ASSET from registry")
114echo "curl -X DELETE https://assets.blockstream.info/$ASSET -H 'Content-Type: application/json' -d '{\"signature\":\"$SIGNED\"}'" > delete_asset_$ASSET.sh
115
116# Stop the daemon
117e1-cli stop
118sleep 10
119
120echo "Completed without error"
