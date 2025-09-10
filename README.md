
ANTONIO PARIENTE GRANERO

https://www.shadertoy.com/view/wflBDr



vec4 circle(vec2 uv, vec2 center, float radius, vec3 color)
{
    float d = length(center - uv) - radius;
    float t = clamp(d, 0.0, 1.0);
    return vec4(color, 1.0 - t);
}

float flowerR(vec2 uv, vec2 center, float baseR, float petals, float phase, float indent)
{
    vec2 p = uv - center;
    float ang = atan(p.y, p.x);
    float w = mix(1.0 - indent, 1.0, 0.5 + 0.5*cos(petals*ang + phase));
    return baseR * w;
}

float seqPetalR(vec2 uv, vec2 center, float baseR, float petals, float phase, float idxf)
{
    vec2 p = uv - center;
    float A = atan(p.y, p.x) + 3.14159265;
    float sec = 6.2831853 / petals;
    float sidx = floor((A + phase) / sec);
    float local = fract((A + phase) / sec);
    float prof = 1.0 - smoothstep(0.28, 0.50, abs(local - 0.5));
    float kf = floor(idxf);
    float gf = fract(idxf);
    float full = step(sidx, kf - 1.0);
    float cur = (1.0 - step(0.5, abs(sidx - kf))) * gf;
    float grow = max(full, cur);
    float r0 = baseR * 0.55;
    float r1 = baseR * (0.95 + 0.20*prof);
    return mix(r0, r1, grow);
}

void mainImage(out vec4 fragColor, in vec2 fragCoord)
{
    vec2 uv = fragCoord.xy;
    vec2 cTop = vec2(iResolution.x*0.5, iResolution.y*0.27);
    vec2 cBot = vec2(iResolution.x*0.5, iResolution.y*0.73);
    vec2 cNew = vec2(iResolution.x*0.82, iResolution.y*0.50);

    vec4 bg = vec4(rgb(255.0, 215.0, 210.0), 1.0);
    vec3 red = rgb(225.0, 50.0, 70.0);
    float baseR = 0.25 * iResolution.y;

    float pulse = 0.9 + 0.1*sin(iTime*2.0);
    float rTop = flowerR(uv, cTop, baseR*pulse, 24.0, 0.0, 0.55);

    float spin = iTime*1.2;
    float rA = flowerR(uv, cBot, baseR, 4.0, spin, 0.45);
    float rB = flowerR(uv, cBot, baseR, 6.0, spin, 0.35);
    float m = 0.5 + 0.5*sin(iTime*0.8);
    float rBot = mix(rA, rB, m);

    float P = 6.0;
    float spd = 0.9;
    float y = mod(iTime*spd, 2.0*P);
    float idxf = (y < P) ? y : (2.0*P - y);

    float baseR2 = 0.22 * iResolution.y;
    float rNewBase = seqPetalR(uv, cNew, baseR2, P, 0.0, idxf);

    float w = 0.45;
    float peak = clamp(1.0 - abs(y - P)/w, 0.0, 1.0);
    float boom = peak*peak*(3.0 - 2.0*peak);
    float rNew = rNewBase * (1.0 + 0.45*boom);

    vec4 top = circle(uv, cTop, rTop, red);
    vec4 bot = circle(uv, cBot, rBot, red);
    vec4 newPetals = circle(uv, cNew, rNew, red);

    vec2 p = uv - cNew;
    float r = length(p);
    float pistR = baseR2 * (0.18 + 0.06*peak);
    vec4 pistil = circle(uv, cNew, pistR, rgb(250.0, 220.0, 90.0));

    float ringR = baseR2 * (0.30 + 0.05*sin(iTime*3.0));
    float outerA = circle(uv, cNew, ringR, vec3(0.0)).a;
    float innerA = circle(uv, cNew, ringR-2.0, vec3(0.0)).a;
    float ringA = max(outerA - innerA, 0.0);
    vec4 ring = vec4(rgb(220.0, 160.0, 60.0), ringA);

    vec4 col = mix(bg, top, top.a);
    col = mix(col, bot, bot.a);
    col = mix(col, newPetals, newPetals.a);
    col = mix(col, pistil, pistil.a);
    col = mix(col, ring, ring.a);

    fragColor = col;
}



