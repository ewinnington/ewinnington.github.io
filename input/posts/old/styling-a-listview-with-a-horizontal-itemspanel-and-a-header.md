Title: Styling a ListView with a Horizontal ItemsPanel and a Header
Published: 23/05/2012
Tags: [Migrated, WPF] 
---

I had to create a specific ListView for my WPF project. A Horizontal alignment of items, with a header column, containing a templated item with a templated checkbox. The final effect looks like this:

[![](old/images/HorizontalListView.png "HorizontalListView")](old/images/HorizontalListView.png)

If we decompose the panel, here are the elements: 

- Black box: ListView with Template containing a Grid with 2 columns 
- Red box: The Header column in the ListView template on Grid.Column="0" 
- Yellow box: The ListView item panel (ItemsPresenter) located on Grid.Column="1" containing an templates ItemsPanelTemplate which is a StackPanel with an Orientation="Horizontal". 
- Green box: The ListView's ItemTemplate which is a StackPanel with an Orientation="Vertical" containing a TextBlock with a RotateTransform LayoutTransform and a Templated Checkbox which uses an Ellipse as it's main shape.

[![](old/images/HorizontalListViewDetails.png "HorizontalListViewDetails")](old/images/HorizontalListViewDetails.png)
```XAML
<ListView ItemsSource="{Binding OpUnitList}" Grid.Row="1" Name="FilterListBox">
 <ListView.ItemsPanel>
   <ItemsPanelTemplate>
     <StackPanel Orientation="Horizontal" IsItemsHost="True">
     </StackPanel>
   </ItemsPanelTemplate>
 </ListView.ItemsPanel>
 <ListView.ItemTemplate>
   <DataTemplate>
     <StackPanel Orientation="Vertical">
       <TextBlock Text="{Binding OpUnitName}" Width="60" Margin="2">
         <TextBlock.LayoutTransform>
           <RotateTransform Angle="-90"/>
         </TextBlock.LayoutTransform>
       </TextBlock>
       <CheckBox IsChecked="{Binding FilterValue}" Name="RgDot" IsThreeState="True" Style="{StaticResource EllipseCheckBoxStyle}"/>
     </StackPanel>
   </DataTemplate>
 </ListView.ItemTemplate>
 <ListView.Template>
   <ControlTemplate>
     <Grid HorizontalAlignment="Left"> 
       <Grid.ColumnDefinitions>
         <ColumnDefinition Width="Auto"></ColumnDefinition>
         <ColumnDefinition Width="\*"></ColumnDefinition>
       </Grid.ColumnDefinitions>
       <StackPanel Orientation="Vertical" Grid.Column="0">
         <TextBlock Margin="2" Height="60">OpUnit Name</TextBlock>
         <TextBlock Margin="2">Filter</TextBlock>
       </StackPanel>
       <ItemsPresenter Grid.Column="1"></ItemsPresenter>
     </Grid>
   </ControlTemplate>
 </ListView.Template>
</ListView>
```