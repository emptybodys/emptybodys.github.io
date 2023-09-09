---
layout:     post
title:      "BirdsSong: A stylized generative audio steganography"
subtitle:   "Sanfeng Zhang, Baiyu Tian, Mengyao Dai, Wang Yang"
date:       2023-9-9
header-img: "img/bg-little-universe.jpg"
---
 
<div>
    <b>Abstract</b>
    <br>StyleGAN and its variants have made significant advancements in the field of image synthesis, allowing for the generation of high-resolution images with precise control over the generation process. Inspired by StyleGAN, this paper introduces a novel audio steganography framework named BirdsSong. The BirdsSong framework aims to enhance steganalysis resistance, increase steganographic capacity, improve audio quality, and diversify the range of styles compared to traditional audio steganography methods. The proposed framework incorporates an audio style extractor that effectively separates the content and style of the reference audio. Additionally, a generator is designed to combine the extracted style vector with the content vector derived from the secret message, resulting in the generation of high-fidelity stego audio. Furthermore, the generator is enhanced to accommodate audio with various styles. Moreover, a new message extractor is devised to recover the secret message with minimal loss in accuracy. Experimental results demonstrate that BirdsSong surpasses baseline methods in terms of steganographic capacity, audio quality, style diversity, and resistance to steganalysis tools.
    <br>
    <br><b>Architecture and Workflow of BirdsSong framework</b>
    <br>As shown in figure, the stylized generative audio steganography framework proposed in this paper consists of a preprocess module, a generator module, and an extractor module.
    <br>
    <img class="shadow" src="/img/in-post/network%20framework.png" width="600">
    <br>Firstly, in the preprocess module, we map the secret message through mapping algorithm into a content vector C that conforms to the normal distribution Z, and extract the stylized feature S1 of the target style audio in the dataset through the WORLD vocoder.
    <br>
    <br>Next, in the generator module, the preprocessed stylized feature S1 is fed into the style extractor S to further extract the advanced style feature vector S2. Subsequently, by employing the generator G, we couple the S2 with the content vector C mapped from the secret message to produce a high-fidelity stego audio. To optimize the generator and the discriminator, the stego audio, the real audio, and the penalty audio synthesized from the two previous audio are fed into the discriminator D separately, resulting in discrimination outcomes Ds, Dr, and Dp. Then the similarity is calculated according to Ds and Dr, and fed back to the generator, and the discriminator loss is calculated according to Ds, Dr and Dp, and fed back to the discriminator. Moreover, the stego audio is passed through the style extractor to extract the style vector S3, S3 is then compared with the advanced style feature S2 extracted from the target style audio, thereby evaluating the style loss throughout the generation process. Finally, the generator adapts its generation capability based on the feedback information encompassing content loss, style loss, and the similarity from the discriminator.
    <br>
    <br>In the message extractor module, we use the message extractor to extract content vector C' from the stego audio, and then convert content vector C' back to Secret message through inverse mapping. Comparing the extracted content vector C' with the original content vector C, we calculate the content loss and fed back to the generator.
    <br>
    <br><b>Preprocess Module</b>
    <br>The main task of the preprocess module is to map the secret message into a content vector conforming to a Gaussian distribution and to extract the style feature vector from the referenced style audio. Specifically, we input the waveform signals of sample audio into the WORLD vocoder to extract fundamental frequency features (F0), spectral envelope features, and aperiodic features, which prepares for the extraction of advanced style features during stego audio generation.
    <br>
    <br>This paper improves the mapping algorithm between secret messages and feature vectors to enhance extraction accuracy. Specifically, we convert the read character string into 8-bit binary according to ASCII code, and combine it into a bit stream. This bit stream is then divided into segments m of length σ, where each segment’s decimal representation falls within the range [0, 2^\sigma-1]. Subsequently, following Equation (1), each segment is mapped into a tensor z. The parameter δ is used to determine the upper and lower boundaries of vector mapping for secret message, while the parameter μ further enhances the fault tolerance space based on δ. It's defined that δ ≤ μ, and the sizes of δ and μ are adjusted as needed to balance extraction accuracy and the quality of stego audio.
    <br>
    <br>  &z=2×m+12σ-1+random(-δ,+δ)&z∈range2×m+12σ-1-μ，2×m+12σ-1+μ&δ≤μ				(1)
    <br>
    <br><b>Generator Module</b>
    <br>The stego audio generator module is the core part of the whole framework. Its primary role is to combine the content vector C obtained in the preprocessing stage with the advanced style feature vector S' extracted by the style extractor to generate the stego audio. Inspired by StyleGAN, we introduce an idea of style-content coupling within the GAN-based audio generative network, achieving both effective transformation outcomes and faster conversion speeds.
    <br>
    <br>We design a style extractor to extract the style feature vector S from the style audio. The structure of the style extractor, as shown in figure, is composed of convolutional layers, residual blocks, and pooling layers. The convolutional layers serve to extract advanced features from the style audio, while the residual blocks enhance the network's expressive capacity and mitigate gradient vanishing issues, and the pooling layer helps the model to focus on the style features and output the result through the linear layer.
    <br>
    <img class="shadow" src="/img/in-post/style%20encoder.png" width="600">
    <br>There is a mapping relationship between the trained style extractor and the audio feature space, which can be represented by Equation (2). Through the mapping relationship, we are able to decouple the audio features into content vectors and style vectors. The dimensionality of the style vectors extracted by the style extractor is [1, 64].
    <br>
    <br>  [Content,Style]sampleStyle ExtractorStyle1×64				(2)
    <br>
    <br>Subsequently, the content vectors and the style vectors are coupled to generate stego audio by the generator. The network structure of the generator is shown as figure, comprises a down-sampling convolutional layer, an up-sampling transpose convolutional layer, and a coupling layer for feature coupling. The down-sampling layer is responsible for extracting advanced features from the input content vector to better integrate with the style features and generate high-fidelity audio. The up-sampling layer restores the coupled features to audio form. The coupling layer helps to better combine the content features and the style features, culminating in the final output through the transpose convolutional layers.
    <br>
    <img class="shadow" src="/img/in-post/gengerator.png" width="600">
    <br>Finally, the stego audio generated by the generator is fed into the discriminator. The discriminator distinguishes between the stego audio and the real audio and outputs a similarity score. The network structure of the discriminator, as shown in figure, employs a fully convolutional layer design to continually extract advanced features and output discrimination results, encouraging the generator to generate high-fidelity audio.
    <br>
    <img class="shadow" src="/img/in-post/discriminator.png" width="600">
    <br><b>Extractor Module</b>
    <br>The message extractor module uses a specially trained extractor to decouple the stego audio into corresponding content vector and style vector. And the content vector is inversely mapped back to the secret message according to Equation (2). To ensure the consistency of the extracted content vector, this method trains the extractor together with the generator and the discriminator to promote the generation of consistent and realistic stego audio.
    <br>
    <br>The network structure of the extractor is shown in figure, consisting of convolutional layers, residual blocks, and transposed convolutional layers. The convolutional layers are used to extract advanced features from decoupled content features, the residual blocks enhance the content vector's extraction capability and accuracy, and the transposed convolutional layers transform the advanced content features back into the secret message vector. Additionally, a concat layer and feature summation operations are incorporated to prevent feature degradation and vanishing issues during training, which effectively preserve content features and improve extraction efficiency.
    <br>
    <img class="shadow" src="/img/in-post/extractor.png" width="600">
    <br><b>Compare And Show Audios</b>
    <br>
    <table border="1">
    <tr>
    <td><b>Real Bird</b></td>
    <td><b>Real Rain</b></td>
    <td><b>BirdsSong Bird</b></td>
    <td><b>BirdsSong Rain</b></td>
    <td><b>Griffin-Lim</b></td>
    <td><b>WaveNet</b></td>
    <td><b>WaveGlow</b></td>
    </tr>
    <tr>
    <td><audio src="/media/real/bird/1.wav" controls="controls" controlsList="nodownload" οncοntextmenu="return false" style="width:50%;"></audio></td>
    <td><audio src="/media/real/rain/1.wav" controls="controls" controlsList="nodownload" οncοntextmenu="return false" style="width:50%;"></audio></td>
    <td><audio src="/media/birdssong/bird/1.wav" controls="controls" controlsList="nodownload" οncοntextmenu="return false" style="width:50%;"></audio></td>
    <td><audio src="/media/birdssong/rain/1.wav" controls="controls" controlsList="nodownload" οncοntextmenu="return false" style="width:50%;"></audio></td>
    </tr>
    <tr>
    <td><audio src="/media/real/bird/2.wav" controls="controls" controlsList="nodownload" οncοntextmenu="return false" style="width:50%;"></audio></td>
    <td><audio src="/media/real/rain/2.wav" controls="controls" controlsList="nodownload" οncοntextmenu="return false" style="width:50%;"></audio></td>
    <td><audio src="/media/birdssong/bird/2.wav" controls="controls" controlsList="nodownload" οncοntextmenu="return false" style="width:50%;"></audio></td>
    <td><audio src="/media/birdssong/rain/2.wav" controls="controls" controlsList="nodownload" οncοntextmenu="return false" style="width:50%;"></audio></td>
    </tr>
    <tr>
    <td><audio src="/media/real/bird/3.wav" controls="controls" controlsList="nodownload" οncοntextmenu="return false" style="width:50%;"></audio></td>
    <td><audio src="/media/real/rain/3.wav" controls="controls" controlsList="nodownload" οncοntextmenu="return false" style="width:50%;"></audio></td>
    <td><audio src="/media/birdssong/bird/3.wav" controls="controls" controlsList="nodownload" οncοntextmenu="return false" style="width:50%;"></audio></td>
    <td><audio src="/media/birdssong/rain/3.wav" controls="controls" controlsList="nodownload" οncοntextmenu="return false" style="width:50%;"></audio></td>
    </tr>
    <tr>
    <td><audio src="/media/real/bird/4.wav" controls="controls" controlsList="nodownload" οncοntextmenu="return false" style="width:50%;"></audio></td>
    <td><audio src="/media/real/rain/4.wav" controls="controls" controlsList="nodownload" οncοntextmenu="return false" style="width:50%;"></audio></td>
    <td><audio src="/media/birdssong/bird/4.wav" controls="controls" controlsList="nodownload" οncοntextmenu="return false" style="width:50%;"></audio></td>
    <td><audio src="/media/birdssong/rain/4.wav" controls="controls" controlsList="nodownload" οncοntextmenu="return false" style="width:50%;"></audio></td>
    </tr>
    <tr>
    <td><audio src="/media/real/bird/5.wav" controls="controls" controlsList="nodownload" οncοntextmenu="return false" style="width:50%;"></audio></td>
    <td><audio src="/media/real/rain/5.wav" controls="controls" controlsList="nodownload" οncοntextmenu="return false" style="width:50%;"></audio></td>
    <td><audio src="/media/birdssong/bird/5.wav" controls="controls" controlsList="nodownload" οncοntextmenu="return false" style="width:50%;"></audio></td>
    <td><audio src="/media/birdssong/rain/5.wav" controls="controls" controlsList="nodownload" οncοntextmenu="return false" style="width:50%;"></audio></td>
    </tr>
    </table>
    
</div>
