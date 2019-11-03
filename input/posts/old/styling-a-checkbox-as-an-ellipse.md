Title: Styling a Checkbox as an Ellipse
Published: 23/05/2012
Tags: [Migrated, WPF] 
---

I wanted to restyle a WPF Checkbox so that is used an Ellipse as a image. It also had to support the IsThreeState property and show a clear sign that the value is null.

[![](old/images/HorizontalListView.png "HorizontalListView")](old/images/HorizontalListView.png)

The image above shows the different styles available (Grey: False, Green: True, White and Barred: Null).

```XAML
<Style x:Key="EllipseCheckBoxStyle" TargetType="CheckBox">
 <Setter Property="SnapsToDevicePixels" Value="true"/>
 <Setter Property="OverridesDefaultStyle" Value="true"/>
 <Setter Property="Template">
   <Setter.Value>
     <ControlTemplate TargetType="CheckBox">
       <Border x:Name="Border" Width="32" Height="32" Background="Transparent" CornerRadius="1" BorderThickness="1" BorderBrush="DarkGray">
         <Grid>
           <Ellipse x:Name="MyEllipse" MinHeight="5" MinWidth="5" Stretch="Fill" Fill="{Binding Path=FilterValue, Converter={StaticResource BoolToRG}}"></Ellipse>
           <Line x:Name="Diag" Stroke="Transparent" X1="0" Y1="0" X2="1" Y2="1" Stretch="Fill"/>
         </Grid>
       </Border>
 
     <ControlTemplate.Triggers>
       <Trigger Property="IsChecked" Value="{x:Null}">
         <Setter TargetName="Diag" Property="Stroke" Value="Black" />
       </Trigger>
       <Trigger Property="IsMouseOver" Value="true">
         <Setter TargetName="Border" Property="Background" Value="Orange" />
       </Trigger>
     </ControlTemplate.Triggers>
   </ControlTemplate>
 </Setter.Value>
 </Setter>
</Style>
```

The converter itself is simply a Bool? to colour converter that I use. You could also set the colour as a Trigger on the control template itself on the IsChecked property like I did for the Null value.
