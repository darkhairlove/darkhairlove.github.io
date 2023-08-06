---
layout: post
title: "[Review]-MusicVAE Paper"
date: 2023-07-04 08:43:59
categories: AI
tags: [Review]
---

## 논문 : [A Hierarchical Latent Vector Model for Learning Long-Term Structure in Music](https://arxiv.org/abs/1803.05428)

### 요약
- 원문 : existing recurrent VAE models have difficulty modeling sequences with long-term structure. To address this issue, we propose the use of a **hierarchical decoder**, which first outputs embeddings for subsequences of the input and then uses these embeddings to generate each subsequence independently. 
This structure encourages the model to utilize its **latent code**, thereby avoiding the “posterior collapse” problem.
this architecture to modeling **sequences of musical notes** and find that it exhibits dramatically better sampling, interpolation, and reconstruction performance than a “flat” baseline model.
- hierarchical decoder을 사용해서 long-term structure에서 어려움을 해결했다.
- 이것은 먼저 입력의 후속에 대한 임베딩을 출력한 다음 이러한 임베딩을 사용하여 각 후속을 독립적으로 생성한다.
- latent code를 활용해 ‘posterior collapse’문제를 피할 수 있었다.
- 음표의 시퀀스 모델링에 적용하고 "flat" baseline 모델보다 더 나은 sampling, interpolation, reconstruction performance을 보여준다.
- 이 논문이 나아가는 방향은 음표의 시퀀스 모델링에 초점을 맞춰져 있다. 그 이유는 대중 음악이 섹션 사이의 반복과 다양성과 같이 strong long-term 구조를 보여주기 때문이고, 이 구조는 hierarchical-songs으로 measures, beats 등으로 나뉘기 때문이다.
- background  
    근본적인 모델은 autoencoder로, 이 논문에서는 추가적으로  perform latent-space interpolations 와 attribute vector arithmetic을 원하여 **VAE**를 사용했다. 
    
    - VAE 이란 ?
    학습 데이터를 병목(Bottleneck)구조를 통해 저차원의 잠재코드(latent code, z)를 만드는 것. 잠재 공간(latent space)을 통해서 모델은 입력 데이터 간의 유사성과 차이를 학습한다.
    VAE는 입력 데이터를 평균, 표준편차 벡터로 인코딩 한 후, 두 벡터에 대응하는 분포에서 샘플링을 수행. 그 후 KL-divergence을 손실함수로 사용해 해당 분포가 표준 정규 분포에 가까워지도록 학습한다.
    결국, 샘플링된 VAE의 잠재 공간 분포는 원점을 기준으로 데이터가 대칭적이고, 분포가 고른 형태를 보인다.  

    [참고링크](https://medium.com/@seyong.dev/vae-variational-auto-encoder-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-60032f19b9e3)  

  - 원문 : Following the framework of **Variational Inference**, we do posterior inference by minimizing the KL divergence between our approximate posterior, the encoder,   
    and the true posterior $$p(z|x)$$ by maximizing the evidence lower bound (ELBO)
        
![스크린샷 2023-07-04 오후 4.31.47.png](/assets/img/content/mu/1.png)
        
  - 변분추론(**Variational Inference**)이란 ?
    posterior $$p(z|x)$$ 분포를 다루기 쉬운 $$q(z)$$로 근사(approximation)하는 것.
    이때, KL-Divergence 사용한다.
    좌변을 ELBO(evidence lower bound)라고 한다. 좌변의 첫째 항인 기댓값은 decoder의 reconstruction 정확도이고, 두번째 항은 KL-Divergence는 $$p(z)$$로부터 잠재 코드(latent code, z)를 샘플링 하여 realistic data를 생성하는 성능으로 0에 가까우면 높은 성능을 보인다.  

    [참고링크](https://ratsgo.github.io/generative%20model/2017/12/19/vi/)
    - $$\beta$$-VAE 와 Free BITS 란 ?
    KL-Divergence 가중치 하이퍼 파라미터 Beta를 사용하는 것.
        
        ![스크린샷 2023-07-04 오후 4.37.55.png](/assets/img/content/mu/2.png)
        
        $$\beta <1$$으로 했을 때, the model to prioritize reconstruction quality over learning a compact representation. 품질을 높여준다. 반대로 샘플링을 통한 잠재 코드의 품질을 높인다.
        
        KL-Divergence에 하한선(free-bits)($$\tau$$)를 정하는 것. freedom을 얻고 reconstruction quality를 높인다.
        
        ![스크린샷 2023-07-04 오후 4.38.50.png](/assets/img/content/mu/3.png)
        
    - Latent Space Manipulation
    원문 : For creative applications, we have additional uses for the latent space learned by the model. 
    선형보간을 사용해서 새로운 잠재 코드를 얻는다.
        
        ![스크린샷 2023-07-04 오후 4.42.41.png](/assets/img/content/mu/4.png)
        
    - Recurrent VAEs
        
        원문 : There are two main drawbacks of this approach. First, RNNs are themselves typically used on their own as powerful autoregressive models of sequences. Second, the model must compress the entire sequence to a single latent vector.
        
        - 단점들 : RNN은 강력한 자기회귀모델으로 RNN 디코더는 데이터 학습할 때, 잠재코드를 무시할 수 있다. 모델은 전체 시퀀스를 단일 잠재 벡터로 압축시켜야 해서, 길어진 시퀀스에서는 실패한다.
        - 따라서 MusicVAE는 계층적 RNN을 디코더에 사용.
1. Model 구조
    
    ![스크린샷 2023-07-04 오후 5.43.31.png](/assets/img/content/mu/5.png)
    
    인코더는 양방향 LSTM, 디코더는 2개 층의 단방향 LSTM
    
    - Bidirectional Encoder
        
        원문 : For the encoder $$q_\lambda(z|x)$$, we use a two-layer bidirectional LSTM network.
        We process an input sequence $$x = \{x_1,x_2,...,x_T\}$$ to obtain the final state vectors $$\overrightarrow h_T, \overleftarrow h_T$$ from the second bidirectional LSTM layer. These are then concatenated to produce $$h_T$$ and fed into two fully connected layers to produce the latent distribution parameters  $$\mu$$ and $$\sigma$$ : where $$W_{h\sigma}$$  ,  $$W_{h\mu}$$  and $$b_{\mu}$$ ,  $$b_{\sigma}$$  are weight matrices and bias vectors, respectively.
        we use an LSTM state size of 2048 for all layers and 512 latent dimensions.
        
        ![스크린샷 2023-07-04 오후 5.11.14.png](/assets/img/content/mu/6.png)
        
        인코더 $$q_\lambda(z|x)$$에 2개 층의 양방향 LSTM 층 사용하고, 2번째 양방향 LSTM층에서 최종 상태 $$\overrightarrow h_T, \overleftarrow h_T$$ 을 얻고, 두 벡터를 concatenation하여 $$h_T$$를 만들어 2개의 fully connected layers를 통과시켜 잠재 분포 파라미터인 $$\mu$$ 와 $$\sigma$$ 를 생성한다.
        이 모델의 state 사이즈는 2048이고 잠재 차원을 512로 두었다.
        
    - Hierarchical Decoder
        
        we found that using a simple RNN as the decoder resulted in poor sampling and reconstruction for long sequences. We believe this is caused by the vanishing influence of the latent state as the output sequence is generated.
        To mitigate this issue, we propose a novel hierarchical RNN for the decoder. Assume that the input sequence (and target output sequence) $$\chi$$ can be segmented into $$U$$ nonoverlapping subsequences $$y_u$$ with endpoints $$i_u$$ so that where we define the special case of $$i_{U +1} = T$$
        
        The conductor RNN produces $$U$$ embedding vectors $$c = \{c_1, c_2, . . . , c_U \}$$, one for each subsequence. In our experiments, we use a two-layer unidirectional LSTM for the conductor with a hidden state size of 1024 and 512 output dimensions.
        
        Once the conductor has produced the sequence of embedding vectors c, each one is individually passed through a shared fully-connected layer followed by a tanh activation to produce initial states for a final bottom-layer decoder RNN. The decoder RNN then autoregressively produces a sequence of distributions over output tokens for each subsequence $$y_u$$ via a softmax output layer. At each step of the bottom-level decoder, the current conductor embedding $$c_u$$  is concatenated with the previous output token to be used as the input.
        
        ![스크린샷 2023-07-04 오후 5.20.47.png](/assets/img/content/mu/7.png)
        
        전체 입력 시퀀스 $$\chi$$를 중복하지 않도로 $$U$$개의 하위 시퀀스 $$y_u$$ 로 나눕니다. $$i_{U +1} = T$$로 정의.
        
        conductor RNN은 $$U$$개의 임베딩 벡터 $$c = \{c_1, c_2, . . . , c_U \}$$를 생성하고, 크기는 1024, 출력 차원은 512로 2개의 층을 가진 단방향 LSTM을 사용한다.
        
        임베딩 벡터 $$c$$를 생성하면 각 벡터는 $$tanh$$ 활성함수가 있는 fully-connected layer를 통과하여 디코더의 bottom-layer의 초기 상태를 생성한다. 그리고 softmax 출력층을 통해 자기회귀적으로 각 subsequence $$y_u$$에 해당하는 분포를 생성한다. 각 bottom-level 디코더에서 현재 conductor 임베딩 $$c_u$$는 이전 출력 토큰과 concatenated 되어 입력에 사용된다.
        
### 마무리
  - long-term구조에서 hierarchical decoder를 사용하는 recurrent VAE인 musicVAE를 제안한다.