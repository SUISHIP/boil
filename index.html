import React, { useState, useEffect, useRef, useCallback } from 'react';
import { Upload, Play, Pause, Download, RefreshCw, Trash2, Settings, Image as ImageIcon, Check, Info } from 'lucide-react';

/**
 * 外部スクリプト(UPNG.js, pako)を読み込むためのカスタムフック
 * エクスポート機能に使用します。
 */
const useExternalScript = (url: string) => {
  const [loaded, setLoaded] = useState(false);
  useEffect(() => {
    const existingScript = document.querySelector(`script[src="${url}"]`);
    if (existingScript) {
      setLoaded(true);
      return;
    }
    const script = document.createElement("script");
    script.src = url;
    script.async = true;
    script.onload = () => setLoaded(true);
    document.body.appendChild(script);
  }, [url]);
  return loaded;
};

// --- 型定義 ---

interface AppState {
  imageLoaded: boolean;
  processing: boolean;
  isPlaying: boolean;
  frames: ImageData[];
  originalImage: HTMLImageElement | null;
  previewCanvasSize: { width: number; height: number };
}

interface BoilParams {
  frameCount: number;
  jitterStrength: number; // 0.0 - 5.0
  noiseScale: number;     // 0.01 - 0.2
  speedFps: number;
  removeWhiteBg: boolean;
  whiteThreshold: number; // 0 - 255
  seed: number;
}

const DEFAULT_PARAMS: BoilParams = {
  frameCount: 3,
  jitterStrength: 2,
  noiseScale: 0.05,
  speedFps: 8,
  removeWhiteBg: false,
  whiteThreshold: 240,
  seed: 12345,
};

// --- ヘルパー関数: ノイズ生成と画像処理 ---

/**
 * 簡易的な疑似乱数生成器 (Seeded Random)
 */
const pseudoRandom = (seed: number) => {
  let value = seed;
  return () => {
    value = (value * 9301 + 49297) % 233280;
    return value / 233280;
  };
};

/**
 * 2D Noise function (簡易版Value Noise)
 * パフォーマンス重視のため、完全なPerlin/Simplexではなく、
 * グリッド補間によるValue Noiseを使用します。
 */
const noise2D = (x: number, y: number, seed: number) => {
    const X = Math.floor(x);
    const Y = Math.floor(y);
    const xf = x - X;
    const yf = y - Y;
    
    const rand = pseudoRandom(seed + X * 57 + Y * 131);
    const tl = rand();
    const tr = rand();
    const bl = rand();
    const br = rand();

    const u = xf * xf * (3 - 2 * xf);
    const v = yf * yf * (3 - 2 * yf);

    return (tl * (1 - u) + tr * u) * (1 - v) + (bl * (1 - u) + br * u) * v;
};

/**
 * 白背景を透明に変換する関数
 * @param imageData 処理対象のImageData
 * @param threshold しきい値 (0-255)
 */
const removeWhiteBackground = (imageData: ImageData, threshold: number) => {
  const data = imageData.data;
  for (let i = 0; i < data.length; i += 4) {
    const r = data[i];
    const g = data[i + 1];
    const b = data[i + 2];
    // RGBすべてがしきい値を超えている場合、透明にする
    if (r > threshold && g > threshold && b > threshold) {
      data[i + 3] = 0;
    }
  }
};

/**
 * ボイルエフェクト（ジッター）を適用したフレームを生成する関数
 * @param ctx Source Context
 * @param width Width
 * @param height Height
 * @param params パラメータ
 * @param frameIndex 現在のフレーム番号（シード変化用）
 */
const generateBoilFrame = (
  originalImageData: ImageData,
  width: number,
  height: number,
  params: BoilParams,
  frameIndex: number
): ImageData => {
  const outputImg = new ImageData(width, height);
  const srcData = originalImageData.data;
  const destData = outputImg.data;
  
  // シード値をフレームごとにずらす
  const frameSeed = params.seed + frameIndex * 1000;

  for (let y = 0; y < height; y++) {
    for (let x = 0; x < width; x++) {
      // ノイズによる変位（Displacement）を計算
      // x, y 座標とスケールに基づいてノイズを取得 (-1.0 ~ 1.0)
      const nx = (noise2D(x * params.noiseScale, y * params.noiseScale, frameSeed) - 0.5) * 2;
      const ny = (noise2D(x * params.noiseScale + 100, y * params.noiseScale + 100, frameSeed) - 0.5) * 2;

      // ジッター強度を適用
      const offsetX = nx * params.jitterStrength;
      const offsetY = ny * params.jitterStrength;

      // 参照元の座標 (整数に丸める)
      const srcX = Math.round(x + offsetX);
      const srcY = Math.round(y + offsetY);

      // 範囲内チェック
      if (srcX >= 0 && srcX < width && srcY >= 0 && srcY < height) {
        const srcIdx = (srcY * width + srcX) * 4;
        const destIdx = (y * width + x) * 4;

        destData[destIdx] = srcData[srcIdx];       // R
        destData[destIdx + 1] = srcData[srcIdx + 1]; // G
        destData[destIdx + 2] = srcData[srcIdx + 2]; // B
        destData[destIdx + 3] = srcData[srcIdx + 3]; // A
      }
    }
  }

  return outputImg;
};

// --- React コンポーネント ---

const App = () => {
  // 外部ライブラリのロード状態
  const pakoLoaded = useExternalScript("https://cdnjs.cloudflare.com/ajax/libs/pako/2.1.0/pako.min.js");
  const upngLoaded = useExternalScript("https://cdnjs.cloudflare.com/ajax/libs/upng-js/2.1.0/UPNG.min.js");

  // State
  const [originalImage, setOriginalImage] = useState<HTMLImageElement | null>(null);
  const [params, setParams] = useState<BoilParams>(DEFAULT_PARAMS);
  const [frames, setFrames] = useState<ImageData[]>([]);
  const [isPlaying, setIsPlaying] = useState(false);
  const [currentFrameIndex, setCurrentFrameIndex] = useState(0);
  const [isProcessing, setIsProcessing] = useState(false);
  const [errorMsg, setErrorMsg] = useState<string | null>(null);

  // Refs
  const canvasRef = useRef<HTMLCanvasElement>(null);
  const containerRef = useRef<HTMLDivElement>(null);
  const fileInputRef = useRef<HTMLInputElement>(null);
  const requestRef = useRef<number>();

  // 画像アップロード処理
  const handleImageUpload = (event: React.ChangeEvent<HTMLInputElement>) => {
    const file = event.target.files?.[0];
    if (!file) return;

    // ファイルサイズチェック (簡易)
    if (file.size > 5 * 1024 * 1024) {
      if(!confirm("画像サイズが大きいため（5MB以上）、処理に時間がかかる可能性があります。続行しますか？")) return;
    }

    const reader = new FileReader();
    reader.onload = (e) => {
      const img = new Image();
      img.onload = () => {
        setOriginalImage(img);
        setFrames([]); // フレームリセット
        setIsPlaying(true);
        setErrorMsg(null);
      };
      img.onerror = () => setErrorMsg("画像の読み込みに失敗しました。");
      img.src = e.target?.result as string;
    };
    reader.readAsDataURL(file);
  };

  // フレーム生成処理 (Effect)
  useEffect(() => {
    if (!originalImage) return;

    const process = async () => {
      setIsProcessing(true);
      
      // UIをブロックしないように少し待つ
      await new Promise(resolve => setTimeout(resolve, 10));

      try {
        const offscreen = document.createElement('canvas');
        const w = originalImage.width;
        const h = originalImage.height;
        offscreen.width = w;
        offscreen.height = h;
        const ctx = offscreen.getContext('2d');
        if (!ctx) throw new Error("Canvas context error");

        // 元画像を描画
        ctx.drawImage(originalImage, 0, 0);
        let baseImageData = ctx.getImageData(0, 0, w, h);

        // 白背景除去が必要なら適用
        if (params.removeWhiteBg) {
          removeWhiteBackground(baseImageData, params.whiteThreshold);
        }

        const newFrames: ImageData[] = [];
        
        // 指定枚数分のフレームを生成
        for (let i = 0; i < params.frameCount; i++) {
          const frame = generateBoilFrame(baseImageData, w, h, params, i);
          newFrames.push(frame);
        }

        setFrames(newFrames);
      } catch (e) {
        console.error(e);
        setErrorMsg("画像処理中にエラーが発生しました。");
      } finally {
        setIsProcessing(false);
      }
    };

    process();
  }, [originalImage, params.frameCount, params.jitterStrength, params.noiseScale, params.removeWhiteBg, params.whiteThreshold, params.seed]);

  // アニメーションループ
  const animate = useCallback(() => {
    if (!canvasRef.current || frames.length === 0) return;
    
    const ctx = canvasRef.current.getContext('2d');
    if (!ctx) return;

    // 現在のフレームを取得して描画
    // 注意: レンダリング時はCanvasサイズに合わせてスケールする必要がある
    const frameData = frames[currentFrameIndex];
    
    // ImageDataを直接putするとサイズが合わないため、
    // 一時的なCanvasを作成してBitmapとして描画する (拡大縮小対応)
    createImageBitmap(frameData).then(bitmap => {
      if (!canvasRef.current) return; // アンマウントチェック
      
      const cvs = canvasRef.current;
      // キャンバスサイズを親コンテナに合わせる（レスポンシブ）
      // しかし、画質維持のため、キャンバスの内部解像度は画像解像度に合わせるのがベスト
      // 表示サイズはCSSで制御
      
      // キャンバスをクリア
      ctx.clearRect(0, 0, cvs.width, cvs.height);
      ctx.drawImage(bitmap, 0, 0);
      bitmap.close();
    });

    if (isPlaying && frames.length > 1) {
       // 次のフレームへ
       const timeoutId = setTimeout(() => {
         setCurrentFrameIndex((prev) => (prev + 1) % frames.length);
         requestRef.current = requestAnimationFrame(animate);
       }, 1000 / params.speedFps);

       // クリーンアップ用
       return () => clearTimeout(timeoutId);
    }
  }, [frames, currentFrameIndex, isPlaying, params.speedFps]);

  // アニメーションのトリガー
  useEffect(() => {
    // 最初の描画
    if (frames.length > 0 && canvasRef.current && originalImage) {
        canvasRef.current.width = originalImage.width;
        canvasRef.current.height = originalImage.height;
    }

    if (isPlaying) {
        const cleanup = animate();
        return () => {
            if (requestRef.current) cancelAnimationFrame(requestRef.current);
            if (cleanup) cleanup();
        };
    }
  }, [isPlaying, animate, frames, originalImage]);


  // エクスポート機能 (APNG)
  const handleExport = async (format: 'APNG' | 'GIF' = 'APNG') => {
    if (frames.length === 0 || !upngLoaded || !pakoLoaded) {
      alert("ライブラリの読み込み中、またはフレームがありません。");
      return;
    }

    setIsProcessing(true);
    await new Promise(r => setTimeout(r, 50)); // UI更新待ち

    try {
      const width = frames[0].width;
      const height = frames[0].height;
      const buffers = frames.map(f => f.data.buffer);
      // UPNG.encode(imgs, w, h, cnum, delays)
      // delays はミリ秒単位
      const delay = 1000 / params.speedFps;
      const delays = new Array(frames.length).fill(delay);

      // @ts-ignore (UPNG is global from script tag)
      const upng = window.UPNG;
      
      let outputData: ArrayBuffer;
      
      // 0 means lossless
      outputData = upng.encode(buffers, width, height, 0, delays);

      const blob = new Blob([outputData], { type: 'image/png' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = `boil_effect_${Date.now()}.png`; // APNGは拡張子.png
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
      URL.revokeObjectURL(url);

    } catch (e) {
      console.error(e);
      alert("エクスポートに失敗しました。");
    } finally {
      setIsProcessing(false);
    }
  };

  const reset = () => {
    setOriginalImage(null);
    setFrames([]);
    setParams(DEFAULT_PARAMS);
  };

  return (
    <div className="min-h-screen bg-slate-50 text-slate-800 font-sans selection:bg-indigo-100">
      {/* Header */}
      <header className="bg-white border-b border-slate-200 px-6 py-4 flex items-center justify-between shadow-sm sticky top-0 z-10">
        <div className="flex items-center gap-2">
          <RefreshCw className="text-indigo-600 animate-spin-slow" size={24} />
          <h1 className="text-xl font-bold bg-clip-text text-transparent bg-gradient-to-r from-indigo-600 to-violet-600">
            Line Boil Animator
          </h1>
        </div>
        <div className="flex gap-2">
           <button 
             onClick={reset}
             disabled={!originalImage}
             className="p-2 text-slate-500 hover:text-red-500 transition-colors disabled:opacity-30"
             title="リセット"
           >
             <Trash2 size={20} />
           </button>
        </div>
      </header>

      <main className="container mx-auto max-w-6xl p-4 lg:p-6 grid grid-cols-1 lg:grid-cols-3 gap-6">
        
        {/* Left Column: Preview Area */}
        <div className="lg:col-span-2 flex flex-col gap-4">
          <div 
            ref={containerRef}
            className="relative w-full aspect-square lg:aspect-video bg-slate-200 rounded-xl overflow-hidden shadow-inner border border-slate-300 flex items-center justify-center group"
            // 市松模様の背景 (透明透過表現)
            style={{
                backgroundImage: `
                  linear-gradient(45deg, #ccc 25%, transparent 25%),
                  linear-gradient(-45deg, #ccc 25%, transparent 25%),
                  linear-gradient(45deg, transparent 75%, #ccc 75%),
                  linear-gradient(-45deg, transparent 75%, #ccc 75%)
                `,
                backgroundSize: '20px 20px',
                backgroundPosition: '0 0, 0 10px, 10px -10px, -10px 0px'
            }}
          >
            {!originalImage ? (
              <div className="text-center p-8 bg-white/80 backdrop-blur-sm rounded-lg shadow-lg">
                <Upload className="mx-auto text-indigo-400 mb-3" size={48} />
                <p className="text-lg font-medium text-slate-700 mb-2">画像をアップロード</p>
                <p className="text-sm text-slate-500 mb-4">PNG, JPG, WEBP (白背景 or 透明)</p>
                <input 
                  type="file" 
                  ref={fileInputRef}
                  onChange={handleImageUpload} 
                  accept="image/png, image/jpeg, image/webp" 
                  className="hidden" 
                />
                <button 
                  onClick={() => fileInputRef.current?.click()}
                  className="bg-indigo-600 hover:bg-indigo-700 text-white px-6 py-2 rounded-full font-medium transition-transform active:scale-95 shadow-md shadow-indigo-200"
                >
                  ファイルを選択
                </button>
              </div>
            ) : (
              <>
                <canvas 
                  ref={canvasRef}
                  className="max-w-full max-h-full object-contain shadow-lg"
                />
                {isProcessing && (
                  <div className="absolute inset-0 bg-black/20 backdrop-blur-[2px] flex items-center justify-center z-20">
                    <div className="bg-white px-6 py-4 rounded-lg shadow-xl flex items-center gap-3">
                      <RefreshCw className="animate-spin text-indigo-600" />
                      <span className="font-medium text-slate-700">処理中...</span>
                    </div>
                  </div>
                )}
                {/* Playback Controls Overlay */}
                <div className="absolute bottom-4 left-1/2 -translate-x-1/2 bg-white/90 backdrop-blur px-4 py-2 rounded-full shadow-lg flex items-center gap-4 opacity-0 group-hover:opacity-100 transition-opacity duration-300">
                    <button onClick={() => setIsPlaying(!isPlaying)} className="hover:text-indigo-600">
                        {isPlaying ? <Pause size={20} fill="currentColor" /> : <Play size={20} fill="currentColor" />}
                    </button>
                    <span className="text-xs font-mono text-slate-500">
                        FRAME: {currentFrameIndex + 1}/{params.frameCount}
                    </span>
                </div>
              </>
            )}
          </div>
          
          {originalImage && (
             <div className="bg-blue-50 text-blue-700 text-sm p-3 rounded-lg flex items-start gap-2 border border-blue-100">
               <Info size={16} className="mt-0.5 shrink-0" />
               <p>
                 エクスポートは <strong>APNG (Animated PNG)</strong> 形式で行われます。
                 GIFよりも高品質で、完全なアルファチャンネル（半透明）を保持できます。
                 主要なブラウザ、LINE、Discord等で動きます。
               </p>
             </div>
          )}
        </div>

        {/* Right Column: Controls */}
        <div className="lg:col-span-1 bg-white rounded-xl shadow-sm border border-slate-200 p-6 flex flex-col h-full overflow-y-auto">
          <div className="flex items-center gap-2 mb-6 text-slate-800 font-semibold border-b pb-2">
            <Settings size={20} />
            <h2>設定パラメータ</h2>
          </div>

          <div className="space-y-6 flex-1">
            {/* Control Group 1: Animation */}
            <div className="space-y-4">
              <label className="block">
                <div className="flex justify-between mb-1">
                  <span className="text-sm font-medium text-slate-700">フレーム数 (Frames)</span>
                  <span className="text-sm text-indigo-600 font-mono">{params.frameCount}</span>
                </div>
                <input 
                  type="range" min="2" max="10" step="1" 
                  value={params.frameCount}
                  onChange={(e) => setParams({...params, frameCount: Number(e.target.value)})}
                  className="w-full accent-indigo-600 h-2 bg-slate-200 rounded-lg appearance-none cursor-pointer"
                />
              </label>

              <label className="block">
                <div className="flex justify-between mb-1">
                  <span className="text-sm font-medium text-slate-700">揺れの強さ (Jitter)</span>
                  <span className="text-sm text-indigo-600 font-mono">{params.jitterStrength}px</span>
                </div>
                <input 
                  type="range" min="0" max="10" step="0.5" 
                  value={params.jitterStrength}
                  onChange={(e) => setParams({...params, jitterStrength: Number(e.target.value)})}
                  className="w-full accent-indigo-600 h-2 bg-slate-200 rounded-lg appearance-none cursor-pointer"
                />
              </label>

              <label className="block">
                <div className="flex justify-between mb-1">
                  <span className="text-sm font-medium text-slate-700">ノイズの細かさ (Scale)</span>
                  <span className="text-sm text-indigo-600 font-mono">{params.noiseScale}</span>
                </div>
                <input 
                  type="range" min="0.01" max="0.5" step="0.01" 
                  value={params.noiseScale}
                  onChange={(e) => setParams({...params, noiseScale: Number(e.target.value)})}
                  className="w-full accent-indigo-600 h-2 bg-slate-200 rounded-lg appearance-none cursor-pointer"
                />
                <p className="text-xs text-slate-400 mt-1">値が大きいほど細かい揺れになります</p>
              </label>

              <label className="block">
                <div className="flex justify-between mb-1">
                  <span className="text-sm font-medium text-slate-700">再生速度 (FPS)</span>
                  <span className="text-sm text-indigo-600 font-mono">{params.speedFps} fps</span>
                </div>
                <input 
                  type="range" min="2" max="24" step="1" 
                  value={params.speedFps}
                  onChange={(e) => setParams({...params, speedFps: Number(e.target.value)})}
                  className="w-full accent-indigo-600 h-2 bg-slate-200 rounded-lg appearance-none cursor-pointer"
                />
              </label>
            </div>

            <hr className="border-slate-100" />

            {/* Control Group 2: Processing */}
            <div className="space-y-4">
              <div className="flex items-center justify-between">
                <label className="flex items-center gap-2 cursor-pointer">
                  <div className={`w-5 h-5 rounded border flex items-center justify-center transition-colors ${params.removeWhiteBg ? 'bg-indigo-600 border-indigo-600' : 'border-slate-300 bg-white'}`}>
                    {params.removeWhiteBg && <Check size={14} className="text-white" />}
                  </div>
                  <input 
                    type="checkbox" 
                    checked={params.removeWhiteBg}
                    onChange={(e) => setParams({...params, removeWhiteBg: e.target.checked})}
                    className="hidden"
                  />
                  <span className="text-sm font-medium text-slate-700">白背景を除去する</span>
                </label>
              </div>

              {params.removeWhiteBg && (
                 <label className="block pl-7 animate-in fade-in slide-in-from-top-2">
                   <div className="flex justify-between mb-1">
                     <span className="text-xs text-slate-500">除去しきい値</span>
                     <span className="text-xs text-slate-500 font-mono">{params.whiteThreshold}</span>
                   </div>
                   <input 
                     type="range" min="200" max="255" step="1" 
                     value={params.whiteThreshold}
                     onChange={(e) => setParams({...params, whiteThreshold: Number(e.target.value)})}
                     className="w-full accent-slate-400 h-1 bg-slate-200 rounded-lg appearance-none cursor-pointer"
                   />
                 </label>
              )}
            </div>
          </div>

          <div className="mt-6 pt-6 border-t border-slate-100">
             <button
               onClick={() => handleExport('APNG')}
               disabled={!originalImage || isProcessing || !upngLoaded}
               className="w-full bg-slate-800 hover:bg-slate-900 text-white py-3 rounded-lg flex items-center justify-center gap-2 font-medium transition-all active:scale-[0.98] disabled:opacity-50 disabled:cursor-not-allowed shadow-lg shadow-slate-200"
             >
               {isProcessing ? (
                 <>
                    <RefreshCw className="animate-spin" size={20} />
                    <span>処理中...</span>
                 </>
               ) : (
                 <>
                    <Download size={20} />
                    <span>APNG で書き出し</span>
                 </>
               )}
             </button>
             {(!upngLoaded) && <p className="text-xs text-center text-red-400 mt-2">エンコーダをロード中...</p>}
             {errorMsg && <p className="text-xs text-center text-red-500 mt-2">{errorMsg}</p>}
          </div>
        </div>
      </main>
      
      <footer className="mt-12 py-6 text-center text-slate-400 text-sm">
        <p>Processed completely locally in your browser.</p>
      </footer>
    </div>
  );
};

export default App;
