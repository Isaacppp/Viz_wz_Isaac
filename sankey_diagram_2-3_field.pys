import numpy as np
import pandas as pd
import plotly.graph_objects as go

def create_sankey(df, origin, destination1, measure, operation, destination2=None):
    if operation == 'sum':
        if destination2:
            df = df.groupby([origin, destination1, destination2])[measure].sum().reset_index()
        else:
            df = df.groupby([origin, destination1])[measure].sum().reset_index()
    elif operation == 'count':
        if destination2:
            df = df.groupby([origin, destination1, destination2]).size().reset_index(name=measure)
        else:
            df = df.groupby([origin, destination1]).size().reset_index(name=measure)

    if destination2:
        label_list = list(set(df[origin].tolist() + df[destination1].tolist() + df[destination2].tolist()))
        source_indices = [label_list.index(o) for o in df[origin]] + [label_list.index(d1) for d1 in df[destination1]]
        target_indices = [label_list.index(d1) for d1 in df[destination1]] + [label_list.index(d2) for d2 in df[destination2]]
    else:
        label_list = list(set(df[origin].tolist() + df[destination1].tolist()))
        source_indices = [label_list.index(o) for o in df[origin]]
        target_indices = [label_list.index(d1) for d1 in df[destination1]]

    # Calculate totals for each node
    totals = np.zeros(len(label_list))
    for s, t, v in zip(source_indices, target_indices, df[measure].tolist() * (2 if destination2 else 1)):
        totals[s] += v
        totals[t] += v

    # Calculate percentages for each node based on the total values for each specific node (origin, destination1, or destination2)
    if destination2:
        total_origin = df.groupby(origin)[measure].sum()
        total_dest1 = df.groupby(destination1)[measure].sum()
        total_dest2 = df.groupby(destination2)[measure].sum()
        for i, label in enumerate(label_list):
            if label in total_origin.index:
                totals[i] = total_origin[label]
            elif label in total_dest1.index:
                totals[i] = total_dest1[label]
            elif label in total_dest2.index:
                totals[i] = total_dest2[label]
    else:
        total_origin = df.groupby(origin)[measure].sum()
        total_dest1 = df.groupby(destination1)[measure].sum()
        for i, label in enumerate(label_list):
            if label in total_origin.index:
                totals[i] = total_origin[label]
            elif label in total_dest1.index:
                totals[i] = total_dest1[label]

    percentages = np.zeros(len(label_list))
    for i, label in enumerate(label_list):
        if label in df[origin].values:
            percentages[i] = totals[i] / total_origin.sum() * 100
        elif label in df[destination1].values:
            percentages[i] = totals[i] / total_dest1.sum() * 100             
        elif destination2 and label in df[destination2].values:
            percentages[i] = totals[i] / total_dest2.sum() * 100

    # Define the colors for the nodes
    origin_color_dict = {"A": "#70C1B3", "B": "#FFE066", "C": "#F25F5C"} # Adjust this according to your actual data
    dest1_color_dict = {"X": "#808F85", "Y": "#91C499", "Z": "#CFD11A"}  # Adjust this according to your actual data
    dest2_color_dict = {"I": "#D87CAC", "II": "#60B2E5", "III": "#53F4FF"}  # Adjust this according to your actual data

    color_dict = {**origin_color_dict, **dest1_color_dict, **dest2_color_dict}  # Combine the dictionaries

    # Extract the node names from the labels
    node_names = []
    for label in label_list:
        match = re.match(r"(.+?) \(", label)
        if match is not None:
            name = match.group(1)
        else:
            name = label  # Use the entire label if no match is found
        node_names.append(name)

    # Assign colors based on the node names
    node_colors = [color_dict.get(name, "#D3D3D3") for name in node_names]

    # Define the colors for the links (based on the color of the origin nodes)
    if destination2:
        link_colors = [f'rgba({",".join(str(int(color_dict.get(src, "#D3D3D3")[1:][i:i+2], 16)) for i in (0, 2, 4))},0.5)' for src in df[origin].tolist() + df[destination1].tolist()]
    else:
        link_colors = [f'rgba({",".join(str(int(color_dict.get(src, "#D3D3D3")[1:][i:i+2], 16)) for i in (0, 2, 4))},0.5)' for src in df[origin].tolist()]

    # Create labels with totals and percentages
    label_list = [f'{label} ({total:,.0f}, {percentage:.2f}%)' for label, total, percentage in zip(label_list, totals, percentages)]

    fig = go.Figure(data=[go.Sankey(
        node=dict(
            pad=15,
            thickness=20,
            line=dict(color="black", width=0.5),
            label=label_list,
            color=node_colors
        ),
        link=dict(
            source=source_indices,
            target=target_indices,
            value=df[measure].tolist() * (2 if destination2 else 1),
            color=link_colors
        )
    )])

    # Update the font here:
    fig.update_traces(textfont=dict(size=20, color='black', family='Bahnschrift SemiBold'))

    
    fig.update_layout(
        title_text="",
        font_size=10,
        width=1000,
        height=800)
    fig.show()

create_sankey(df5, 'source', 'dest1', 'weight', 'sum', 'dest2')
