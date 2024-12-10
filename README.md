# team_41_automated-text-summarization 

The motive of this project is about building an automated text summarizer that takes input texts and outputs corredsponding summaries particulary it processes the human speech to create the input texts after deploying the model. The model that was built was of Transformer architecture that has custom multi-layered encoders that work on the outputs of preceding encoder and the starting first level encoder takes input texts alongside the custom multi-layered encoders we have custom multi-layered decoders that takes keys,values of the last layer of the multi-layered encoder and processes it with the queries of the output summaries while training and calculating loss and updating the parameters based on backpropogation from loss.
So, first the dataset of format .csv was loaded in the jupyter notebook. The dataset structure was id,dialogues,summaries,topics. The columns id,topics weren't considered while preprocessing. So now after finding out that 97 percent of the dialogue texts have the text length below 1584 where 1584 is the maximum text length that lies in that 97 percent of the text dialogues. And the same criteria was seen for summaries and maximum summary text length was observed to be 283. So the dialogue texts and summary texts with size less than 1584 and size less that 290 respectively will be furthur processed. Now a new dataset will be create with dialogue texts and their corresponding summaries and the the dataset will be loaded as batches of batch size 2 using DataLoader.
Now to tokenize the texts a transformer based tokenizer T5Tokenizer was used which would tokenize texts and does padding by padding tokens and adds start and end tokens and the for each text it creates a list of token_ids in sequence which can be acesed by tokenizer(text)['input_ids'] but before all this the maximum number of tokens in the input dialogue text and output summary texts were checked and found to be 573 and 129 respectively so the maximum length for can be initialized padding. This kind of padding ensures that the number of input words that the encoder can take always remains constant and same for the decoder with output summary words. But the padding tokens will be ignored when we create padding masks for encoder and decoder like 1 padding mask for each encoder layer and 2 padding masks for each deocder layer one for supressing output padded tokens and another near the cross-attention layer where the keys and values are taken from according to the output of the encoder.
And then tensors input_ids and output_ids were created for each tensor batch where each batch has 2 dialogue texts where each dialogue text has a specific tokenid embeddings and output_ids are tensors where each tensor batch has 2 corresponding summary text that has specific tokenid embeddings for 2 dialogue texts in input_ids tensors.
And then all the input_ids tensors and output_ids tensors in input_ids and output_ids are concatenated or stacked as large tensor tensor_tokenized_inputs(of size = 12085*573) and tensor_tokenized_outputs(of size = 12085*150)
And before further processsing the custom multi-layered encoders and custom multi-layered decoders were costructed inspired from the pipeline of the transformer:
Encoder: 
  1.Multiheadattention: After the input positional word embeddings has come as input in the form X then we create 3 matrices Q,K,V using nn.Linear and the reshaping,permuting(interchanging dimensions) and chunking it into 3 matrices Q,K,V and calculate the self attention by the fourmula product = (Q.K_transpose/sqrt(dk))where dk is the number of dimensions of Q,K,V at each head.Then the product is then masked where the padding tokens will be ignored and will become zero when passed in softmax function attention_scores = softmax(product) is calculated along dimension -1 and the attention_scores are multiplied with V and after this is calculated for each head the outputs from each head will be concatenated and passed into a linear layer and returned.
  2.Then few neurons are dropped from the outputs from Multiheadattention Layer with probability 0.1.
  3.Then the residual from the after adding positinal encoding layer is added to this outputs to deal with vanishing gradients problem after dropping few neurons from outputs for more generalization then this addition is passed to Layernormalisation to help the model in stable training as it is easy to take ven steps during learning process.
  4. Then the residual the input of Layernormalisation is added to output of Layernormalisation and passed to feed foward(Linear,ReLu,dropout) after which few dropouts happen based on probability 0.1 and residual the input of feed foward is added to the feed foward output and then this added output is again perfrming Layernormalisation.
  5. The output of one encoder is passed to subsequent encoder layer and so on and the output of last encoder layer is passed to decoder at its cross attention layer where new K,V are created based on the it but the Q vector will be created based on the inputs summary text scentences word embeddings + posotional encodings that are given to decoder. 
  6. In decoder the Multiheadmasked self attention will calculate attention scores only with the values that lies within the index of query vector all other attention scores with values outside query vector index will be producted to be zero. it's like we are masking the future so the model does not see the future and learns by itself by learning from losses and backpropogating and updating parameters and predicts the next words by its own.
  rest everything in decoder is almost similar to encoder except the multihead cross attention which was explained in point 6.
  And the process down to this was about creation of word embeddings from conversion of tensor_tokenized_inputs,tensor_tokenized_outputs to one-hot-vectors to word embeddings and adding them with positional embeddings(Built using sin and cos)  according to batch size 2 such that the parameters will be updated after processing a batch according to by backpropogating from the loss.
Building the transformer model by syncronizing encoder and decoder layers,computing loss,backpropogation,updating parameters will be dne by next submission.



hello doggy