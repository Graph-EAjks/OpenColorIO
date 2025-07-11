..
  SPDX-License-Identifier: CC-BY-4.0
  Copyright Contributors to the OpenColorIO Project.
  Do not edit! This file was automatically generated by share/docs/frozendoc.py.

.. py:class:: GpuShaderDesc
   :module: PyOpenColorIO
   :canonical: PyOpenColorIO.GpuShaderDesc

   This class holds the GPU-related information needed to build a shader program from a specific processor.

   This class defines the interface and there are two implementations provided. The "legacy" mode implements the OCIO v1 approach of baking certain ops in order to have at most one 3D-LUT. The "generic" mode is the v2 default and allows all the ops to be processed as-is, without baking, like the CPU renderer. Custom implementations could be written to accommodate the GPU needs of a specific client app.

   The complete fragment shader program is decomposed in two main parts: the OCIO shader program for the color processing and the client shader program which consumes the pixel color processing.

   The OCIO shader program is fully described by the :ref:`GpuShaderDesc` independently from the client shader program. The only critical point is the agreement on the OCIO function shader name.

   To summarize, the complete shader program is:

   .. code-block:: cpp


       //                                                                    //
       //               The complete fragment shader program                 //
       //                                                                    //
       //                                                                    //
       //   //////////////////////////////////////////////////////////////   //
       //   //                                                          //   //
       //   //               The OCIO shader program                    //   //
       //   //                                                          //   //
       //   //////////////////////////////////////////////////////////////   //
       //   //                                                          //   //
       //   //   // All global declarations                             //   //
       //   //   uniform sampled3D tex3;                                //   //
       //   //                                                          //   //
       //   //   // All helper methods                                  //   //
       //   //   vec3 computePos(vec3 color)                            //   //
       //   //   {                                                      //   //
       //   //      vec3 coords = color;                                //   //
       //   //      ...                                                 //   //
       //   //      return coords;                                      //   //
       //   //   }                                                      //   //
       //   //                                                          //   //
       //   //   // The OCIO shader function                            //   //
       //   //   vec4 OCIODisplay(in vec4 inColor)                      //   //
       //   //   {                                                      //   //
       //   //      vec4 outColor = inColor;                            //   //
       //   //      ...                                                 //   //
       //   //      outColor.rbg                                        //   //
       //   //         = texture3D(tex3, computePos(inColor.rgb)).rgb;  //   //
       //   //      ...                                                 //   //
       //   //      return outColor;                                    //   //
       //   //   }                                                      //   //
       //   //                                                          //   //
       //   //////////////////////////////////////////////////////////////   //
       //                                                                    //
       //   //////////////////////////////////////////////////////////////   //
       //   //                                                          //   //
       //   //             The client shader program                    //   //
       //   //                                                          //   //
       //   //////////////////////////////////////////////////////////////   //
       //   //                                                          //   //
       //   //   uniform sampler2D image;                               //   //
       //   //                                                          //   //
       //   //   void main()                                            //   //
       //   //   {                                                      //   //
       //   //      vec4 inColor = texture2D(image, gl_TexCoord[0].st); //   //
       //   //      ...                                                 //   //
       //   //      vec4 outColor = OCIODisplay(inColor);               //   //
       //   //      ...                                                 //   //
       //   //      gl_FragColor = outColor;                            //   //
       //   //   }                                                      //   //
       //   //                                                          //   //
       //   //////////////////////////////////////////////////////////////   //
       //                                                                    //


   **Usage Example:** *Building a GPU shader*

   This example is based on the code in: src/apps/ociodisplay/main.cpp

   .. code-block:: cpp

       // Get the processor
       //
       OCIO::ConstConfigRcPtr config = OCIO::Config::CreateFromEnv();
       OCIO::ConstProcessorRcPtr processor
          = config->getProcessor("ACES - ACEScg", "Output - sRGB");

       // Step 1: Create a GPU shader description
       //
       // The two potential scenarios are:
       //
       //   1. Instantiate the generic shader description.  The color processor
       //      is used as-is (i.e. without any baking step) and could contain
       //      any number of 1D & 3D luts.
       //
       //      This is the default OCIO v2 behavior and allows a much better
       //      match between the CPU and GPU renderers.
       //
       OCIO::GpuShaderDescRcPtr shaderDesc = OCIO::GpuShaderDesc::Create();
       //
       //   2. Instantiate a custom shader description.
       //
       //      Writing a custom shader description is a way to tailor the shaders
       //      to the needs of a given client program.  This involves writing a
       //      new class inheriting from the pure virtual class GpuShaderDesc.
       //
       //      Please refer to the GenericGpuShaderDesc class for an example.
       //
       OCIO::GpuShaderDescRcPtr shaderDesc = MyCustomGpuShader::Create();

       shaderDesc->setLanguage(OCIO::GPU_LANGUAGE_GLSL_1_2);
       shaderDesc->setFunctionName("OCIODisplay");

       // Step 2: Collect the shader program information for a specific processor
       //
       processor->extractGpuShaderInfo(shaderDesc);

       // Step 3: Create a helper to build the shader. Here we use a helper for
       //         OpenGL but there will also be helpers for other languages.
       //
       OpenGLBuilderRcPtr oglBuilder = OpenGLBuilder::Create(shaderDesc);

       // Step 4: Allocate & upload all the LUTs
       //
       oglBuilder->allocateAllTextures();

       // Step 5: Build the complete fragment shader program using
       //         g_fragShaderText which is the client shader program.
       //
       g_programId = oglBuilder->buildProgram(g_fragShaderText);

       // Step 6: Enable the fragment shader program, and all needed textures
       //
       glUseProgram(g_programId);
       glUniform1i(glGetUniformLocation(g_programId, "tex1"), 1);  // image texture
       oglBuilder->useAllTextures(g_programId);                    // LUT textures

       // Step 7: Update uniforms from dynamic property instances.
       m_oglBuilder->useAllUniforms();


   .. py:method:: GpuShaderDesc.CreateShaderDesc(language: PyOpenColorIO.GpuLanguage = <GpuLanguage.GPU_LANGUAGE_GLSL_1_2: 1>, functionName: str = 'OCIOMain', pixelName: str = 'outColor', resourcePrefix: str = 'ocio', uid: str = '') -> PyOpenColorIO.GpuShaderDesc
      :module: PyOpenColorIO
      :staticmethod:

      Create the default shader description.


   .. py:attribute:: GpuShaderDesc.TEXTURE_1D
      :module: PyOpenColorIO
      :value: <TextureDimensions.TEXTURE_1D: 1>


   .. py:attribute:: GpuShaderDesc.TEXTURE_2D
      :module: PyOpenColorIO
      :value: <TextureDimensions.TEXTURE_2D: 2>


   .. py:class:: GpuShaderDesc.TextureDimensions
      :module: PyOpenColorIO
      :canonical: PyOpenColorIO.GpuShaderCreator.TextureDimensions

      Dimension enum used to differentiate between 1D and 2D object/resource types.

      Members:

        TEXTURE_1D

        TEXTURE_2D


      .. py:attribute:: GpuShaderDesc.TextureDimensions.TEXTURE_1D
         :module: PyOpenColorIO
         :value: <TextureDimensions.TEXTURE_1D: 1>


      .. py:attribute:: GpuShaderDesc.TextureDimensions.TEXTURE_2D
         :module: PyOpenColorIO
         :value: <TextureDimensions.TEXTURE_2D: 2>


      .. py:method:: GpuShaderDesc.TextureDimensions.__init__(self: PyOpenColorIO.GpuShaderCreator.TextureDimensions, value: int) -> None
         :module: PyOpenColorIO


      .. py:property:: GpuShaderDesc.TextureDimensions.name
         :module: PyOpenColorIO


      .. py:property:: GpuShaderDesc.TextureDimensions.value
         :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.__init__(*args, **kwargs)
      :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.add3DTexture(self: PyOpenColorIO.GpuShaderDesc, textureName: str, samplerName: str, edgeLen: int, interpolation: PyOpenColorIO.Interpolation, values: Buffer) -> None
      :module: PyOpenColorIO

      Add a 3D texture with RGB channel type.

      .. note::
         The 'values' parameter contains the 3D LUT data which must be used as-is as the dimension and origin are hard-coded in the fragment shader program. So, it means one GPU 3D texture per entry.


   .. py:method:: GpuShaderDesc.addTexture(self: PyOpenColorIO.GpuShaderDesc, textureName: str, samplerName: str, width: int, height: int, channel: PyOpenColorIO.GpuShaderCreator.TextureType, dimensions: PyOpenColorIO.GpuShaderCreator.TextureDimensions, interpolation: PyOpenColorIO.Interpolation, values: Buffer) -> None
      :module: PyOpenColorIO

      Add a 1D or 2D texture

      .. note::
         The 'values' parameter contains the LUT data which must be used as-is as the dimensions and origin are hard-coded in the fragment shader program. So, it means one GPU texture per entry.


   .. py:method:: GpuShaderDesc.addToDeclareShaderCode(self: PyOpenColorIO.GpuShaderCreator, shaderCode: str) -> None
      :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.addToFunctionFooterShaderCode(self: PyOpenColorIO.GpuShaderCreator, shaderCode: str) -> None
      :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.addToFunctionHeaderShaderCode(self: PyOpenColorIO.GpuShaderCreator, shaderCode: str) -> None
      :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.addToFunctionShaderCode(self: PyOpenColorIO.GpuShaderCreator, shaderCode: str) -> None
      :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.addToHelperShaderCode(self: PyOpenColorIO.GpuShaderCreator, shaderCode: str) -> None
      :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.begin(self: PyOpenColorIO.GpuShaderCreator, uid: str) -> None
      :module: PyOpenColorIO

      Start to collect the shader data.


   .. py:method:: GpuShaderDesc.clone(self: PyOpenColorIO.GpuShaderDesc) -> PyOpenColorIO.GpuShaderCreator
      :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.createShaderText(self: PyOpenColorIO.GpuShaderCreator, shaderDeclarations: str, shaderHelperMethods: str, shaderFunctionHeader: str, shaderFunctionBody: str, shaderFunctionFooter: str) -> None
      :module: PyOpenColorIO

      Create the OCIO shader program.

      .. note::
         The OCIO shader program is decomposed to allow a specific implementation to change some parts. Some product integrations add the color processing within a client shader program, imposing constraints requiring this flexibility.


   .. py:method:: GpuShaderDesc.end(self: PyOpenColorIO.GpuShaderCreator) -> None
      :module: PyOpenColorIO

      End to collect the shader data.


   .. py:method:: GpuShaderDesc.finalize(self: PyOpenColorIO.GpuShaderCreator) -> None
      :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.get3DTextures(self: PyOpenColorIO.GpuShaderDesc) -> PyOpenColorIO.GpuShaderDesc.Texture3DIterator
      :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.getAllowTexture1D(self: PyOpenColorIO.GpuShaderCreator) -> bool
      :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.getCacheID(self: PyOpenColorIO.GpuShaderCreator) -> str
      :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.getDynamicProperties(self: PyOpenColorIO.GpuShaderCreator) -> PyOpenColorIO.GpuShaderCreator.DynamicPropertyIterator
      :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.getDynamicProperty(self: PyOpenColorIO.GpuShaderCreator, type: PyOpenColorIO.DynamicPropertyType) -> PyOpenColorIO.DynamicProperty
      :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.getFunctionName(self: PyOpenColorIO.GpuShaderCreator) -> str
      :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.getLanguage(self: PyOpenColorIO.GpuShaderCreator) -> PyOpenColorIO.GpuLanguage
      :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.getNextResourceIndex(self: PyOpenColorIO.GpuShaderCreator) -> int
      :module: PyOpenColorIO

      To avoid global texture sampler and uniform name clashes always append an increasing index to the resource name.


   .. py:method:: GpuShaderDesc.getPixelName(self: PyOpenColorIO.GpuShaderCreator) -> str
      :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.getResourcePrefix(self: PyOpenColorIO.GpuShaderCreator) -> str
      :module: PyOpenColorIO

      .. note::
         Some applications require that textures, uniforms, and helper methods be uniquely named because several processor instances could coexist.


   .. py:method:: GpuShaderDesc.getShaderText(self: PyOpenColorIO.GpuShaderDesc) -> str
      :module: PyOpenColorIO

      Get the complete OCIO shader program.


   .. py:method:: GpuShaderDesc.getTextureMaxWidth(self: PyOpenColorIO.GpuShaderCreator) -> int
      :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.getTextures(self: PyOpenColorIO.GpuShaderDesc) -> PyOpenColorIO.GpuShaderDesc.TextureIterator
      :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.getUniforms(self: PyOpenColorIO.GpuShaderDesc) -> PyOpenColorIO.GpuShaderDesc.UniformIterator
      :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.getUniqueID(self: PyOpenColorIO.GpuShaderCreator) -> str
      :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.hasDynamicProperty(self: PyOpenColorIO.GpuShaderCreator, type: PyOpenColorIO.DynamicPropertyType) -> bool
      :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.setAllowTexture1D(self: PyOpenColorIO.GpuShaderCreator, allowed: bool) -> None
      :module: PyOpenColorIO

      Allow 1D GPU resource type, otherwise always using 2D resources for 1D LUTs.


   .. py:method:: GpuShaderDesc.setFunctionName(self: PyOpenColorIO.GpuShaderCreator, name: str) -> None
      :module: PyOpenColorIO


   .. py:method:: GpuShaderDesc.setLanguage(self: PyOpenColorIO.GpuShaderCreator, language: PyOpenColorIO.GpuLanguage) -> None
      :module: PyOpenColorIO

      Set the shader program language.


   .. py:method:: GpuShaderDesc.setPixelName(self: PyOpenColorIO.GpuShaderCreator, name: str) -> None
      :module: PyOpenColorIO

      Set the pixel name variable holding the color values.


   .. py:method:: GpuShaderDesc.setResourcePrefix(self: PyOpenColorIO.GpuShaderCreator, prefix: str) -> None
      :module: PyOpenColorIO

      Set a prefix to the resource name.


   .. py:method:: GpuShaderDesc.setTextureMaxWidth(self: PyOpenColorIO.GpuShaderCreator, maxWidth: int) -> None
      :module: PyOpenColorIO

      Some graphic cards could have 1D & 2D textures with size limitations.


   .. py:method:: GpuShaderDesc.setUniqueID(self: PyOpenColorIO.GpuShaderCreator, uid: str) -> None
      :module: PyOpenColorIO


.. py:class:: TextureType
   :module: PyOpenColorIO.GpuShaderDesc
   :canonical: PyOpenColorIO.GpuShaderCreator.TextureType

   Members:

     TEXTURE_RED_CHANNEL

     TEXTURE_RGB_CHANNEL


   .. py:attribute:: TextureType.TEXTURE_RED_CHANNEL
      :module: PyOpenColorIO.GpuShaderDesc
      :value: <TextureType.TEXTURE_RED_CHANNEL: 0>


   .. py:attribute:: TextureType.TEXTURE_RGB_CHANNEL
      :module: PyOpenColorIO.GpuShaderDesc
      :value: <TextureType.TEXTURE_RGB_CHANNEL: 1>


   .. py:property:: TextureType.value
      :module: PyOpenColorIO.GpuShaderDesc


.. py:class:: UniformData
   :module: PyOpenColorIO.GpuShaderDesc
   :canonical: PyOpenColorIO.GpuShaderDesc.UniformData


   .. py:method:: UniformData.getBool(self: PyOpenColorIO.GpuShaderDesc.UniformData) -> bool
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:method:: UniformData.getDouble(self: PyOpenColorIO.GpuShaderDesc.UniformData) -> float
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:method:: UniformData.getFloat3(self: PyOpenColorIO.GpuShaderDesc.UniformData) -> Annotated[list[float], FixedSize(3)]
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:method:: UniformData.getVectorFloat(self: PyOpenColorIO.GpuShaderDesc.UniformData) -> numpy.ndarray
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:method:: UniformData.getVectorInt(self: PyOpenColorIO.GpuShaderDesc.UniformData) -> numpy.ndarray
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:property:: UniformData.type
      :module: PyOpenColorIO.GpuShaderDesc


.. py:class:: Texture
   :module: PyOpenColorIO.GpuShaderDesc
   :canonical: PyOpenColorIO.GpuShaderDesc.Texture


   .. py:property:: Texture.channel
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:property:: Texture.dimensions
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:method:: Texture.getValues(self: PyOpenColorIO.GpuShaderDesc.Texture) -> numpy.ndarray
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:property:: Texture.height
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:property:: Texture.interpolation
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:property:: Texture.samplerName
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:property:: Texture.textureName
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:property:: Texture.width
      :module: PyOpenColorIO.GpuShaderDesc


.. py:class:: Texture3D
   :module: PyOpenColorIO.GpuShaderDesc
   :canonical: PyOpenColorIO.GpuShaderDesc.Texture3D


   .. py:property:: Texture3D.edgeLen
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:method:: Texture3D.getValues(self: PyOpenColorIO.GpuShaderDesc.Texture3D) -> numpy.ndarray
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:property:: Texture3D.interpolation
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:property:: Texture3D.samplerName
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:property:: Texture3D.textureName
      :module: PyOpenColorIO.GpuShaderDesc


.. py:class:: UniformIterator
   :module: PyOpenColorIO.GpuShaderDesc
   :canonical: PyOpenColorIO.GpuShaderDesc.UniformIterator


   .. py:method:: UniformIterator.__getitem__(self: PyOpenColorIO.GpuShaderDesc.UniformIterator, arg0: int) -> tuple
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:method:: UniformIterator.__iter__(self: PyOpenColorIO.GpuShaderDesc.UniformIterator) -> PyOpenColorIO.GpuShaderDesc.UniformIterator
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:method:: UniformIterator.__len__(self: PyOpenColorIO.GpuShaderDesc.UniformIterator) -> int
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:method:: UniformIterator.__next__(self: PyOpenColorIO.GpuShaderDesc.UniformIterator) -> tuple
      :module: PyOpenColorIO.GpuShaderDesc


.. py:class:: TextureIterator
   :module: PyOpenColorIO.GpuShaderDesc
   :canonical: PyOpenColorIO.GpuShaderDesc.TextureIterator


   .. py:method:: TextureIterator.__getitem__(self: PyOpenColorIO.GpuShaderDesc.TextureIterator, arg0: int) -> PyOpenColorIO.GpuShaderDesc.Texture
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:method:: TextureIterator.__iter__(self: PyOpenColorIO.GpuShaderDesc.TextureIterator) -> PyOpenColorIO.GpuShaderDesc.TextureIterator
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:method:: TextureIterator.__len__(self: PyOpenColorIO.GpuShaderDesc.TextureIterator) -> int
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:method:: TextureIterator.__next__(self: PyOpenColorIO.GpuShaderDesc.TextureIterator) -> PyOpenColorIO.GpuShaderDesc.Texture
      :module: PyOpenColorIO.GpuShaderDesc


.. py:class:: Texture3DIterator
   :module: PyOpenColorIO.GpuShaderDesc
   :canonical: PyOpenColorIO.GpuShaderDesc.Texture3DIterator


   .. py:method:: Texture3DIterator.__getitem__(self: PyOpenColorIO.GpuShaderDesc.Texture3DIterator, arg0: int) -> PyOpenColorIO.GpuShaderDesc.Texture3D
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:method:: Texture3DIterator.__iter__(self: PyOpenColorIO.GpuShaderDesc.Texture3DIterator) -> PyOpenColorIO.GpuShaderDesc.Texture3DIterator
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:method:: Texture3DIterator.__len__(self: PyOpenColorIO.GpuShaderDesc.Texture3DIterator) -> int
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:method:: Texture3DIterator.__next__(self: PyOpenColorIO.GpuShaderDesc.Texture3DIterator) -> PyOpenColorIO.GpuShaderDesc.Texture3D
      :module: PyOpenColorIO.GpuShaderDesc


.. py:class:: DynamicPropertyIterator
   :module: PyOpenColorIO.GpuShaderDesc
   :canonical: PyOpenColorIO.GpuShaderCreator.DynamicPropertyIterator


   .. py:method:: DynamicPropertyIterator.__getitem__(self: PyOpenColorIO.GpuShaderCreator.DynamicPropertyIterator, arg0: int) -> PyOpenColorIO.DynamicProperty
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:method:: DynamicPropertyIterator.__iter__(self: PyOpenColorIO.GpuShaderCreator.DynamicPropertyIterator) -> PyOpenColorIO.GpuShaderCreator.DynamicPropertyIterator
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:method:: DynamicPropertyIterator.__len__(self: PyOpenColorIO.GpuShaderCreator.DynamicPropertyIterator) -> int
      :module: PyOpenColorIO.GpuShaderDesc


   .. py:method:: DynamicPropertyIterator.__next__(self: PyOpenColorIO.GpuShaderCreator.DynamicPropertyIterator) -> PyOpenColorIO.DynamicProperty
      :module: PyOpenColorIO.GpuShaderDesc

