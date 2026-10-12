# watermark-attack-benchmark

Embeds a watermark into image patches with five transform-domain methods (DCT, DWT, SVD, DWT+DCT, DWT+SVD), runs six common attacks on the results, and measures what survives.

Built with: Python, OpenCV, NumPy, PyWavelets, scikit-image, scikit-learn, SHAP · Context: VIT Digital Watermarking and Steganography course project, 2025

![Original patch and the same patch watermarked with each method](Figure_1.png)

## Pipeline

1. `code/preprocess.py` cuts source images into 256×256 patches and splits them 80/20 into train/val.
2. `code/watermark/{dct,dwt,svd,dwt_dct,dwt_svd}.py` embed `watermark.png` (64×64, alpha 0.5) into the luminance channel.
3. `code/attacks_with_metrics.py` applies JPEG (quality 50 and 70), Gaussian noise, Gaussian blur, an 80% centre crop resized back, and a 15° rotation.
4. `code/metrics_with_ber.py` computes PSNR, SSIM and BER for every attacked image.
5. `code/plot_attack_metrics.py` writes the boxplots and heatmaps in `results/`.
6. `code/xai_pipeline.py` extracts simple image features (intensity, entropy, edge density, texture), trains a random forest to predict BER, and explains it with SHAP.

## Results

Mean over 2,500 patches per cell, as PSNR (dB) / SSIM. Both compare each watermarked image to its attacked version, so they show how much each attack distorts the image (higher means a milder attack), not how visible the watermark is.

| Method | Blur | Crop | JPEG50 | JPEG70 | Noise | Rotate |
|---|---|---|---|---|---|---|
| DCT | 30.36 / 0.888 | 16.73 / 0.366 | 32.92 / 0.938 | 34.85 / 0.959 | 23.39 / 0.622 | 16.16 / 0.339 |
| DWT | 33.51 / 0.899 | 18.45 / 0.425 | 33.81 / 0.943 | 35.51 / 0.961 | 23.55 / 0.616 | 17.90 / 0.400 |
| SVD | 24.48 / 0.747 | 14.88 / 0.275 | 28.39 / 0.891 | 30.41 / 0.929 | 23.99 / 0.770 | 14.53 / 0.264 |
| DWT+DCT | 29.71 / 0.884 | 16.87 / 0.377 | 32.61 / 0.935 | 34.39 / 0.955 | 23.38 / 0.616 | 16.28 / 0.350 |
| DWT+SVD | 29.44 / 0.832 | 16.71 / 0.278 | 31.84 / 0.915 | 33.51 / 0.941 | 23.40 / 0.665 | 16.24 / 0.265 |

Source: `results/attack_summary.csv`. Boxplots and heatmaps are in `results/`.

**Known issue:** the extractors in `code/metrics_with_ber.py` compare against a 32×32 watermark, but embedding uses 64×64. Because of that mismatch, the BER values in `dataset/metrics_with_ber.csv` sit around 0.5 (chance level) and don't say anything about robustness yet. The random forest and SHAP step is trained on those BER values, so it has the same problem.

## How to run

The image dataset isn't in the repo. Put source images in `raw_images/` and install the dependencies:

```bash
pip install opencv-python numpy pandas PyWavelets scikit-image scikit-learn shap matplotlib seaborn tqdm joblib
```

Then run the steps above in order. The scripts use relative paths, so run each one from its own folder (`code/` or `code/watermark/`).
