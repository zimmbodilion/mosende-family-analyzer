# Mosende Family Tree Algorithm: ID-based Relationship Analysis
import re
import csv
from typing import List, Dict
import streamlit as st
import graphviz

def parse_id(person_id: str) -> List[int]:
    if '-' in person_id:
        return list(map(int, person_id.split('-')))
    return auto_parse_numeric_id(person_id)

def auto_parse_numeric_id(numeric_id: str) -> List[int]:
    return list(map(int, list(str(numeric_id))))

def generation_depth(person_id: str) -> int:
    return len(parse_id(person_id))

def common_ancestor(id1: str, id2: str) -> str:
    path1 = parse_id(id1)
    path2 = parse_id(id2)
    i = 0
    while i < min(len(path1), len(path2)) and path1[i] == path2[i]:
        i += 1
    return '-'.join(map(str, path1[:i])) if i > 0 else None

def relationship_distance(id1: str, id2: str) -> int:
    path1 = parse_id(id1)
    path2 = parse_id(id2)
    common = parse_id(common_ancestor(id1, id2))
    return (len(path1) - len(common)) + (len(path2) - len(common))

def consanguinity_level(id1: str, id2: str) -> str:
    path1 = parse_id(id1)
    path2 = parse_id(id2)
    common_depth = len(parse_id(common_ancestor(id1, id2)))
    gen1 = len(path1) - common_depth
    gen2 = len(path2) - common_depth
    cousin_level = min(gen1, gen2)
    removal = abs(gen1 - gen2)
    if cousin_level == 0:
        if removal == 0:
            return "Same person"
        else:
            return f"Direct line ({removal} generations apart)"
    else:
        return f"{ordinal(cousin_level)} cousin{' once removed' if removal == 1 else f' {removal} times removed' if removal > 1 else ''}"

def ordinal(n):
    return "%d%s" % (n, "tsnrhtdd"[(n//10%10!=1)*(n%10<4)*n%10::4])

def get_birth_order(person_id: str) -> int:
    return int(parse_id(person_id)[-1])

def infer_affinity_link(id1: str, spouse_map: Dict[str, str]) -> str:
    return spouse_map.get(id1, None)

def export_to_csv(data: List[Dict[str, str]], filename: str):
    if not data:
        print("No data to export.")
        return
    with open(filename, mode='w', newline='', encoding='utf-8') as file:
        writer = csv.DictWriter(file, fieldnames=data[0].keys())
        writer.writeheader()
        writer.writerows(data)
    print(f"Data exported to {filename}")

def render_ancestor_tree(person_id: str):
    path = parse_id(person_id)
    dot = graphviz.Digraph()
    curr_id = ""
    for i, num in enumerate(path):
        curr_id = f"{curr_id}-{num}" if curr_id else str(num)
        dot.node(curr_id, f"Gen {i+1}: {curr_id}")
        if i > 0:
            prev_id = '-'.join(curr_id.split('-')[:-1])
            dot.edge(prev_id, curr_id)
    return dot

# Web Interface using Streamlit
def run_web_app():
    st.title("Mosende Family Tree Analyzer")

    id1 = st.text_input("Enter first person ID (e.g. 1-1-4-6):")
    id2 = st.text_input("Enter second person ID (e.g. 1-1-5-3):")

    if id1 and id2:
        st.write("**Common Ancestor:**", common_ancestor(id1, id2))
        st.write("**Generational Distance:**", relationship_distance(id1, id2))
        st.write("**Consanguinity Level:**", consanguinity_level(id1, id2))

        st.subheader("Ancestor Tree of Person 1")
        st.graphviz_chart(render_ancestor_tree(id1))

        st.subheader("Ancestor Tree of Person 2")
        st.graphviz_chart(render_ancestor_tree(id2))

if __name__ == "__main__":
    # Command line test
    peterjhun = "1-1-4-6-8-2-1"
    vedasto = "1-1-4"
    jose = "1-1-4-6"
    felix = "1-1-5-3"
    edencio = "1-3-1-7"
    raw_id = "1146821"

    print("Generational depth of Peterjhun:", generation_depth(peterjhun))
    print("Common ancestor of Peterjhun and Vedasto:", common_ancestor(peterjhun, vedasto))
    print("Distance (steps) between Peterjhun and Vedasto:", relationship_distance(peterjhun, vedasto))
    print("Consanguinity of Jose and Felix:", consanguinity_level(jose, felix))
    print("Consanguinity of Jose and Edencio:", consanguinity_level(jose, edencio))
    print("Birth order of Peterjhun:", get_birth_order(peterjhun))
    print("Raw numeric ID parsing:", parse_id(raw_id))

    spouse_links = {"1-1-4-6": "1-1-5-3"}
    print("Spouse of Jose:", infer_affinity_link("1-1-4-6", spouse_links))

    # Export sample data to CSV
    data = [
        {"ID1": jose, "ID2": felix, "CommonAncestor": common_ancestor(jose, felix),
         "Distance": relationship_distance(jose, felix), "Consanguinity": consanguinity_level(jose, felix)},
        {"ID1": jose, "ID2": edencio, "CommonAncestor": common_ancestor(jose, edencio),
         "Distance": relationship_distance(jose, edencio), "Consanguinity": consanguinity_level(jose, edencio)}
    ]
    export_to_csv(data, "mosende_relationships.csv")

    # Launch Streamlit if needed
    # run_web_app()
