# README

최근작업한 내용인 Raytracing 과 Footprint 기능에 대한 설명 입니다.

작업내용은 모드 UE4를 수정하여 구현되었습니다.

## 인게임 구현 결과
-   `Raytracing`
    -   Reflection & Global illumination
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/RTXAnimation.gif?raw=true" width=600 height=350>
    -   ParticleSystem with HybridRendering
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/RTXParticleAnimation.gif?raw=true" width=600 height=350>

-   `Footprint`
    -   Character Footprint
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FootprintAnimation.gif?raw=true" width=600 height=350>

## Raytracing
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

## Footprint
-   `구현`   
    -   UE4 ShadowDepthRendering 코드를 참조하여 구현.
    -   Footprint를 남기는 FootprintCaster와 Footprint가 표면에 남게 되는 FootprintReceiver로 구분.
    -   Footprint영역(AABBox)에 들어온 FootprintCaster와 FootprintReceiver 컬링.
        -   최적화를 위해 Octree와 Multithread(PC에선 Parallel For 활용, Mobile에선 MultiThread 사용하지 않음)활용.
    -   FootprintCaster와 FootprintReceiver를 2Pass로 나누어 렌더링.
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FootprintExp.png?raw=true" width=246 height=167>
        -   Depth & Stencil Test를 이용하여 FootprintCaster와 FootprintReceiver가 겹치는 영역 마스킹.
    -  마스킹결과를 이전 프레임의 FootprintMaskBuffer와 블렌딩.
         -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FootprintMask.png?raw=true" width=246 height=167>
    -  현재 프레임의 FootprintMaskBuffer를 이용하여 FootprintTangentBuffer 생성
         -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FootprintNormalMap.png?raw=true" width=246 height=167>
    -  FootprintMaskBuffer와 FootprintTangentBuffer를 사용하여 최종 결과물 생성.    
    - 결과
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/Footprint.jpg?raw=true" width=246 height=167>
