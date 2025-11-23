// ... (前のコードはそのまま) ...

            // Mix
            vec3 col = getPalette(liquid + u_bass * 0.2, u_paletteVal, u_bass, u_mid);
            
            // 【修正1】構造色の倍率を下げる (1.5 -> 1.0)
            vec3 structCol = vec3(0.0, 0.8, 1.0) * (0.5 + u_bass*0.5) * (1.0 + u_paletteVal);
            col = mix(col, structCol * 1.0, structure * 0.7); 
            
            // 【修正2】パーティクルの加算倍率を下げる (2.0 -> 1.2くらいに)
            // これで光るけど白飛びしにくくなる
            col += vec3(particles * 1.2) * structCol; 

            // Post Processing
            vec3 finalCol = col;
            float aber = u_vol * u_react * 0.02;
            finalCol.r = mix(col.r, col.r * 1.1, aber);
            
            float satFactor = smoothstep(0.0, 0.2, u_paletteVal);
            float gray = dot(finalCol, vec3(0.299, 0.587, 0.114));
            float noirContrast = 1.0 + (1.0 - satFactor) * 0.5; 
            vec3 noirGray = vec3(pow(gray, noirContrast));
            finalCol = mix(noirGray, finalCol, satFactor);

            float vig = 1.0 - smoothstep(0.3, 1.8, length(vTexCoord - 0.5) * 1.5);
            finalCol *= vig;
            finalCol += (hash(uv + u_time) - 0.5) * 0.05;

            // 【修正3】トーンマッピングの係数を変更 (0.6 -> 1.2)
            // 数字を大きくすると、強い光をより強く圧縮して白飛びを防ぐんや
            finalCol = finalCol / (finalCol + vec3(1.2)); 
            
            finalCol = pow(finalCol, vec3(1.0/1.8)); 

            gl_FragColor = vec4(finalCol, 1.0);
        }
    `;
