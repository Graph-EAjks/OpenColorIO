..
  SPDX-License-Identifier: CC-BY-4.0
  Copyright Contributors to the OpenColorIO Project.
  Do not edit! This file was automatically generated by share/docs/frozendoc.py.

.. py:class:: DynamicProperty
   :module: PyOpenColorIO
   :canonical: PyOpenColorIO.DynamicProperty

   Allows transform parameter values to be set on-the-fly (after finalization). For example, to modify the exposure in a viewport. Dynamic properties can be accessed from the `:ref:`CPUProcessor`` or `:ref:`GpuShaderCreator`` to change values between processing.

   .. code-block:: cpp

       OCIO::ConstConfigRcPtr config = OCIO::GetCurrentConfig();
       OCIO::ConstProcessorRcPtr processor = config->getProcessor(colorSpace1, colorSpace2);
       OCIO::ConstCPUProcessorRcPtr cpuProcessor = processor->getDefaultCPUProcessor();

       if (cpuProcessor->hasDynamicProperty(OCIO::DYNAMIC_PROPERTY_EXPOSURE))
       {
           // Get the in-memory implementation of the dynamic property.
           OCIO::DynamicPropertyRcPtr dynProp =
               cpuProcessor->getDynamicProperty(OCIO::DYNAMIC_PROPERTY_EXPOSURE);
           // Get the interface used to change the double value.
           OCIO::DynamicPropertyDoubleRcPtr exposure =
               OCIO::DynamicPropertyValue::AsDouble(dynProp);
           // Update of the dynamic property instance with the new value.
           exposure->setValue(1.1f);
       }
       if (cpuProcessor->hasDynamicProperty(OCIO::DYNAMIC_PROPERTY_GRADING_PRIMARY))
       {
           OCIO::DynamicPropertyRcPtr dynProp =
               cpuProcessor->getDynamicProperty(OCIO::DYNAMIC_PROPERTY_GRADING_PRIMARY);
           OCIO::DynamicPropertyGradingPrimaryRcPtr primaryProp =
               OCIO::DynamicPropertyValue::AsGradingPrimary(dynProp);
           OCIO::GradingPrimary primary = primaryProp->getValue();
           primary.m_saturation += 0.1f;
           primaryProp->setValue(primary);
       }
       if (cpuProcessor->hasDynamicProperty(OCIO::DYNAMIC_PROPERTY_GRADING_RGBCURVE))
       {
           OCIO::DynamicPropertyRcPtr dynProp =
               cpuProcessor->getDynamicProperty(OCIO::DYNAMIC_PROPERTY_GRADING_RGBCURVE);
           OCIO::DynamicPropertyGradingRGBCurveRcPtr rgbCurveProp =
               OCIO::DynamicPropertyValue::AsGradingRGBCurve(dynProp);
           OCIO::ConstGradingRGBCurveRcPtr rgbCurve = rgbCurveProp->getValue()->createEditableCopy();
           OCIO::GradingBSplineCurveRcPtr rCurve = rgbCurve->getCurve(OCIO::RGB_RED);
           rCurve->getControlPoint(1).m_y += 0.1f;
           rgbCurveProp->setValue(rgbCurve);
       }


   .. py:method:: DynamicProperty.__init__(*args, **kwargs)
      :module: PyOpenColorIO


   .. py:method:: DynamicProperty.getDouble(self: PyOpenColorIO.DynamicProperty) -> float
      :module: PyOpenColorIO

      Get the property as DynamicPropertyDoubleRcPtr to access the double value. Will throw if property type is not a type that holds a double such as DYNAMIC_PROPERTY_EXPOSURE.


   .. py:method:: DynamicProperty.getGradingPrimary(self: PyOpenColorIO.DynamicProperty) -> PyOpenColorIO.GradingPrimary
      :module: PyOpenColorIO

      Get the property as DynamicPropertyGradingPrimaryRcPtr to access the :ref:`GradingPrimary` value. Will throw if property type is not DYNAMIC_PROPERTY_GRADING_PRIMARY.


   .. py:method:: DynamicProperty.getGradingRGBCurve(self: PyOpenColorIO.DynamicProperty) -> PyOpenColorIO.GradingRGBCurve
      :module: PyOpenColorIO

      Get the property as DynamicPropertyGradingRGBCurveRcPtr to access the GradingRGBCurveRcPtr value. Will throw if property type is not DYNAMIC_PROPERTY_GRADING_RGBCURVE.


   .. py:method:: DynamicProperty.getGradingTone(self: PyOpenColorIO.DynamicProperty) -> PyOpenColorIO.GradingTone
      :module: PyOpenColorIO

      Get the property as DynamicPropertyGradingToneRcPtr to access the :ref:`GradingTone` value. Will throw if property type is not DYNAMIC_PROPERTY_GRADING_TONE.


   .. py:method:: DynamicProperty.getType(self: PyOpenColorIO.DynamicProperty) -> PyOpenColorIO.DynamicPropertyType
      :module: PyOpenColorIO


   .. py:method:: DynamicProperty.setDouble(self: PyOpenColorIO.DynamicProperty, val: float) -> None
      :module: PyOpenColorIO

      Get the property as DynamicPropertyDoubleRcPtr to access the double value. Will throw if property type is not a type that holds a double such as DYNAMIC_PROPERTY_EXPOSURE.


   .. py:method:: DynamicProperty.setGradingPrimary(self: PyOpenColorIO.DynamicProperty, val: PyOpenColorIO.GradingPrimary) -> None
      :module: PyOpenColorIO

      Get the property as DynamicPropertyGradingPrimaryRcPtr to access the :ref:`GradingPrimary` value. Will throw if property type is not DYNAMIC_PROPERTY_GRADING_PRIMARY.


   .. py:method:: DynamicProperty.setGradingRGBCurve(self: PyOpenColorIO.DynamicProperty, val: PyOpenColorIO.GradingRGBCurve) -> None
      :module: PyOpenColorIO

      Get the property as DynamicPropertyGradingRGBCurveRcPtr to access the GradingRGBCurveRcPtr value. Will throw if property type is not DYNAMIC_PROPERTY_GRADING_RGBCURVE.


   .. py:method:: DynamicProperty.setGradingTone(self: PyOpenColorIO.DynamicProperty, val: PyOpenColorIO.GradingTone) -> None
      :module: PyOpenColorIO

      Get the property as DynamicPropertyGradingToneRcPtr to access the :ref:`GradingTone` value. Will throw if property type is not DYNAMIC_PROPERTY_GRADING_TONE.

