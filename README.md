# README

최근작업한 내용인 Raytracing 과 Footprint 기능에 대한 설명 입니다.

## Raytracing
-   `Reflection & Global illumination`
<img src="https://github.com/djarksnd/RecentWork/blob/main/images/RTXAnimation.gif?raw=true" width=600 height=350>

-   `ParticleSystem with HybridRendering`
<img src="https://github.com/djarksnd/RecentWork/blob/main/images/RTXParticleAnimation.gif?raw=true" width=600 height=350>

## Footprint
-   `Character Footprint`
<img src="https://github.com/djarksnd/RecentWork/blob/main/images/FootprintAnimation.gif?raw=true" width=600 height=350>

## Important Classes
-   `TiledRenderer`
    -   GeometryPass, LightPass, ShadowDepthBuffer 클래스 들을 멤버로 소유 하며, 랜더링의 주체로 랜더링 흐름과 랜더링 스래드를 관리 합니다.
    -   주요 코드는 아래의 함수에서 확인하실 수 있습니다.
        -   TiledRenderer.cpp
            -   TiledRenderer::Render
            -   TiledRenderer::FlushRenderTasks
            -   TiledRenderer::RenderingThreadProc
-   `GeometryPass`
    -   씬에 존재하는 오브젝트들의 기하정보를 2개의 [GeometryBuffer](https://en.wikipedia.org/wiki/Glossary_of_computer_graphics#g-buffer)(_DiffuseSpecular_, _NormalGlossiness_)에 기록 합니다.
        -   DiffuseSpecular [R8G8B8A8]
            -   RGB = DiffuseColor
            -   A = Specular intensity
        -   NormalGlossiness [R10G10B10A2]
            -   RG = WorldNormal.XY
            -   B = Glossiness
            -   A = Sign of WorldNormal.Z
    -   주요 코드는 아래의 함수에서 확인하실 수 있습니다.
        -   GeometryPassPixelShader.hlsl
            -   main
-   `LightPass`
    -   ComputeShader를 통해 여러개의 격자로 분할된 화면 공간에 영향을 미치는 조명들의 색인을 계산합니다.
    -   주요 코드는 아래의 함수에서 확인하실 수 있습니다.
        - LightCullingComputeShader.hlsl
            -   main  
        - LightPass.cpp
            -   LightPass::CullLights
-   `ShadowDepthBuffer`
    -   화면에 표시될 그림자들의 ShadowDepthMap을 랜더링 합니다.
    -   PointLight의 그림자를 표현하기 위해 필요한 6면의 ShadowDepthMap을 GeometryShader와 RenderTargetArray를 이용해 One - Pass로 랜더링 합니다.
    -   주요 코드는 아래의 함수에서 확인하실 수 있습니다.
        -   ShadowDepthGeometryShader.hlsl
            -   main
        -   ShadowDepthBuffer.cpp
            -   ShadowDepthBuffer::RenderPointLightShadowDepth

## Files
이 포트폴리오는 실행파일을 빌드 할 수 있는 솔루션 파일과(Visual Studio 2019) 빌드된 바이너리 파일을 포함합니다.

-   `bin` _실행 가능한 바이너리 파일 폴더_
    -   `TiledLighting_x64_release.exe` _실행 가능한 바이너리 파일(Windows 64bit only / 바이너리 파일 실행 시 추가 DLL이 필요할 수 있습니다.)_
-   `dxut` _DXUT프로젝트 폴더 (포트폴리오 빌드 시 필요)_
-   `media` _포트폴리오 데이터 폴더(쉐이더 코드 & 그래픽 데이터)_
-   `TiledLighting` _C++ 소스코드 및 프로젝트 파일 폴더_
-   `TiledLighting_VS2019_Win10.sln` _Visual Studio 2019 솔루션 파일_

## Download Release
<img src="https://github.com/djarksnd/MultiThreadedTiledLighting/blob/master/ScreenShot.png?raw=true" width=400 height=300>

-   빌드과정 없이 포트폴리오를 실행하려면 아래 링크된 파일을 내려받으시면 됩니다.
    -   릴리즈된 바이너리는 Windows 64bit 전용 입니다.
    -   내려받은 파일의 압축을 푼 후 bin폴더의 TiledLighting_x64_release.exe를 실행하세요.
        -   카메라 이동은 a, s, d, w 키를 사용하세요.
        -   카메라 회전은 마우스 좌버튼을 누른 채 마우스를 회전하세요.
    -   MultiThreadedTiledLighting.zip
        -   [https://github.com/djarksnd/MultiThreadedTiledLighting/releases/download/1.0/MultiThreadedTiledLighting.zip](https://github.com/djarksnd/MultiThreadedTiledLighting/releases/download/1.0/MultiThreadedTiledLighting.zip)
