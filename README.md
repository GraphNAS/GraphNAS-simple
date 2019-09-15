# GraphNAS

#### Overview
This directory contains code necessary to run the GraphNAS algorithm. Graph Neural Architecture Search method (GraphNAS 
for short) enables automatic design of the best graph neural architecture based on reinforcement learning. 
Specifically, GraphNAS first uses a recurrent network to generate variable-length strings that describe the 
architectures of graph neural networks and then trains the recurrent network with a policy gradient algorithm 
to maximize the expected accuracy of the generated architectures on a validation data set.

<p align="center">
<img src="./images/macro_search.png" width="500"  alt="A simple illustration of GraphNAS" align=center>
</p>
An illustration of GraphNAS. A recurrent network (Controller RNN) generates descriptions of graph neural architectures (Child model GNNs). 
Once an architecture m is generated by the controller, GraphNAS trains m on a given graph G and test m on a validate set D. 
The validation result R is taken as the reward of the recurrent network.

Furthermore, to improve the search efficiency of GraphNAS on big networks, we restrict the search space from an entire 
architecture to a sequential concatenation of the best search results built on each single architecture layer.

<p align="center">
<img src="./images/micro_search.png" width="500"  alt="A simple illustration of GraphNAS" align=center>
</p>
An illustration of GraphNAS constructing  a single GNN layer at the right-hand side. 
The layer has two input states O_1 and O_2, two intermediate states O_3 and O_4, and an output state O_5. 
The controller at the left-hand side  samples O_2 from {O_1, O_2, O_3} and take O_2 as the input  of O_4, 
and then samples "GAT" for processing O_2.  
The output state O_5=relu(O_3+O_4) collects information from  O_3 and O_4, 
and the controller assigns a readout operator "add" and an activation operator "relu" for O_5. 
As a result, this layer can be described as a list of operators: [0, gcn, 1, gat, add, relu].


#### Requirements
Recent versions of PyTorch, numpy, scipy, sklearn, dgl, torch_geometric and networkx are required.
Ensure that at least PyTorch 1.0.0 is installed. Then run:
    
    pip install -r requirements.txt

If you want to run in docker, you can run:

    docker build -t graphnas -f DockerFile . \
    docker run -it -v $(pwd):/GraphNAS graphnas python main.py --dataset cora

#### Running the code
##### Architecture evaluation
To evaluate our best architecture found in semi-supervised experiments by training from scratch, run

    python -m eval_scripts.semi.eval_found_gnn

To evaluate our best architecture found in semi-supervised experiments by training from scratch, run

    python -m eval_scripts.sup.eval_found_gnn
###### Results
Semi-supervised node classification w.r.t. accuracy

Model| Cora | Citeseer | Pubmed
|-|-|-|-| 
GCN    | 81.5+/-0.4 | 70.9+/-0.5   | 79.0+/-0.4  
SGC    |  81.0+/-0.0 |   71.9+/-0.1   |  78.9+/-0.0   
GAT    |  83.0+/-0.7  |  72.5+/-0.7   | 79.0+/-0.3    
LGCN    |  83.3+/-0.5  | 73.0+/-0.6    |  79.5+/-0.2   
DGCN    |  82.0+/-0.2  | 72.2+/-0.3    |  78.6+/-0.1   
ARMA    |  82.8+/-0.6  | 72.3+/-1.1    |  78.8+/-0.3   
APPNP   |  83.3+/-0.6  | 71.8+/-0.4    |  80.2+/-0.2   
simple-NAS |  81.4+/-0.6  |  71.7+/-0.6    |  79.5+/-0.5  
GraphNAS | 84.4+/-0.4  | 73.5+/-0.3    | 80.5+/-0.3  
		
Supervised node classification w.r.t. accuracy	

Model| Cora | Citeseer | Pubmed  
|-|-|-|-| 
GCN    | 90.2+/-0.0  | 80.0+/-0.3   | 87.8+/-0.2  
SGC    | 88.8+/-0.0 |  80.6+/-0.0   |   86.5+/-0.1  
GAT    |  89.5+/-0.3  |  78.6+/-0.3    |  86.5+/-0.6   
LGCN    | 88.7+/-0.5  | 79.2+/-0.4     |  OOM    
DGCN    |  88.4+/-0.2  |  78.0+/-0.2    |  88.0+/-0.9    
ARMA    |  89.8+/-0.1  |  79.9+/-0.6    |  88.1+/-0.2    
APPNP    | 90.4+/-0.2  | 79.2+/-0.4     | 87.4+/-0.3    
random-NAS | 90.0+/-0.3   |  81.1+/-0.3    | 90.7+/-0.6    
simple-NAS | 90.1+/-0.3  |  79.6+/-0.5    |  88.5+/-0.2  
GraphNAS | 90.6+/-0.3   |  81.2+/-0.5   | 91.2+/-0.3    
	
Supervised learning on randomly split training data

Model| Cora | Citeseer | Pubmed  
|-|-|-|-| 
GCN    | 88.3+/-1.3 | 77.2+/-1.7   | 88.1+/-1.4  
SGC    | 88.2+/-1.4  |  77.4+/-1.8   |  85.8+/-1.2      
GAT    | 87.2+/-1.1   |   77.1+/-1.3   | 87.8+/-1.4    
LGCN    | 87.9+/-1.5   |   76.6+/-1.6   | OOM    
DGCN    | 87.8+/-1.3  |   74.4+/-1.7   |  88.4+/-1.2    
ARMA    | 88.2+/-1.0  |   76.7+/-1.5   |  88.7+/-1.0    
APPNP    | 87.5+/-1.4   |  77.3+/-1.6    |  88.2+/-1.1    
random-NAS |  88.5+/-1.0  |  76.5+/-1.3    | 90.3+/-0.8   
simple-NAS |  88.5+/-1.0  |   77.5+/-2.3   |  88.5+/-1.1    
GraphNAS | 88.9+/-1.2  |  77.6+/-1.5    |  91.1+/-1.0

Architectures designed in supervised learning are showed as follow:
<p align="center">
<img src="./images/GraphNAS_cells.png" width="500"  alt="Architectures designed by GraphNAS in supervised experiments" align=center>
</p>
The architecture G-Cora designed by GraphNAS on Cora is [0, gat6, 0, gcn, 0, gcn, 2, arma, tanh, concat], 
the architecture G-Citeseer designed by GraphNAS on  Citeseer is [0, identity, 0, gat6, linear, concat], 
the architecture G-Pubmed designed by GraphNAS on  Pubmed is [1, gat8, 0, arma, tanh, concat]. 
##### Searching for new architectures
To carry out architecture search using search space described in Section 3.2, run

    python -m models.common.common_main --dataset Citeseer

To carry out architecture search using search space described in Section 3.4, run
    
    python -m models.micro_nas.micro_main --dataset Citeseer 



#### Acknowledgements
This repo is modified based on [DGL](https://github.com/dmlc/dgl) and [PYG](https://github.com/rusty1s/pytorch_geometric).
