# Seeds & Peers for Laozi mainnet
Your node needs to know how to find peers.  
You'll need to add either seeds (suggested) or persistent peers from the lists below to `$HOME/.band/config/config.toml`.
## Seeds
- Band Protocol  
`2ace5785ad45be3d49eabd937892fab097a3515e@49.12.222.89:26656`  
`12879758fb38f5f80e05022f1102aecebcb56bb0@37.27.249.86:26656`  
- Active Nodes
`2c884e60a0944958a2a9389f07f2f66dcfc3add0@seeds-bandprotocol.activenodes.io:36656`  

## Peers
- Band Protocol (Provided snapshot for state sync every 20k blocks) 
`98823087b61d442a4ab86998709c77b2e517ee78@35.240.152.216:26656`  
`3ea84babead3d6bc488810a0f2cf0744cf5c68fe@34.86.22.251:26656`  
- WeStaking
`d047cfabdfd5e244af530d6d2101d07c45ff7424@165.22.167.234:41656`  
- SolidOne
`1a4af7cbd3db94a3881dc35cfa261ec2ac788f8f@91.246.64.247:26656`  
`c1b93580023f12891683431e25fb05d6e4372bef@188.34.207.243:26656`  
`2f62588be3cf41a019289097c2068c4e65aa5e37@95.217.213.135:26656`  
`58ce8a2bd441057c0b18d0812f692b2d8af55af7@168.119.61.205:26656`  
- Cosmostation
`570787c6484fb5aef9182f032dbd54042d93b93c@35.82.85.220:26656`  
`cb777d12ec515884f9f8153e2bbe1844079323a4@44.234.200.116:26656`  
`cacb630ef586ffefccf103d442c02bdbc187f996@3.37.82.201:26656`  
- Chorus One
`39d45dae55f36db42ef3997376efbbf725666f75@51.38.53.4:30656`  
`70c542ec2e1aebb385f7e93d1c55f1a4073cdcb7@54.38.46.124:30656`  
`d7e04f00e2701cb9b2c62a78b2e7bb3cdb165dfb@51.91.67.48:30656`  

## To get seeds/persistent_peers
`cat | grep '@' | xargs | sed 's/ /,/g'`  
Paste seeds or peers and "CTRL+D"
